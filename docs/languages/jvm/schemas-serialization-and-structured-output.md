# Schemas, Serialization, and Structured Output

## Contracts at every boundary

There are three different contracts:

1. provider-supported structured output;
2. tool/MCP JSON Schema;
3. internal persisted event schema.

Do not assume one generated schema is valid for all three. Providers and SDKs support subsets; MCP Java 2.x validates Draft 2020-12 tool inputs; durable stores require long-lived evolution rules.

## Canonical workflow

~~~mermaid
flowchart LR
    T[Typed domain model] --> G[Schema generation/review]
    G --> P[Provider-compatible schema]
    G --> M[MCP/tool schema]
    G --> E[Persisted event schema]
    R[Untrusted JSON] --> V[Size and syntax limits]
    V --> S[Schema validation]
    S --> D[Typed decode]
    D --> B[Business validation]
~~~

Schema validation cannot express all business rules. Check cross-field invariants, authorization, freshness, resource ownership, and effect limits after decoding.

## Java choices

The official OpenAI Java SDK can derive structured-output schemas from Java classes, but generated schemas still need golden tests against provider restrictions. Jackson 3 is the recommended major for new projects as of the research date; Jackson 2 remains widely used and maintained. They use different packages/group IDs and are not drop-in replacements. Follow the framework BOM rather than forcing a mixed major.

Safe defaults:

- deserialize into narrow DTOs, not <code>Object</code> graphs;
- reject or deliberately capture unknown fields at trust boundaries;
- cap nesting, string, number, collection, and total byte sizes;
- use explicit allowlisted polymorphic subtypes;
- do not enable global default typing for untrusted input;
- never use native Java serialization for provider/tool payloads.

Allowing unknown fields improves forward compatibility but can hide misspellings or ignored security controls. Decide per boundary: strict for executable tool arguments; version-tolerant for read-only provider events when unknown fields are retained diagnostically.

## Kotlin choices

<code>kotlinx.serialization</code> has strict unknown-key handling by default. Configure <code>ignoreUnknownKeys</code>, <code>explicitNulls</code>, and coercion deliberately; permissive settings can weaken validation. Kotlin default values and nullable fields encode different meanings—missing, explicit null, and defaulted must not be conflated for patch/effect requests.

Register polymorphic subclasses in a <code>SerializersModule</code>. This allowlist is a useful security property. Java reflection <code>Type</code> interop can lose Kotlin nullability, so prefer reified/explicit Kotlin serializers for Kotlin models.

For an executable boundary, make the strictness local and visible instead of changing one global `Json` instance used by tolerant provider-event readers:

~~~kotlin
@Serializable
data class TransferArgs(
    val accountId: String,
    val amountMinor: Long,
    val currency: String,
    val reason: String? = null
)

private val toolJson = Json {
    ignoreUnknownKeys = false
    isLenient = false
    coerceInputValues = false
    explicitNulls = true
}

fun decodeTransfer(bytes: ByteArray): TransferArgs =
    toolJson.decodeFromString<TransferArgs>(bytes.decodeToString()).also { args ->
        require(args.accountId.matches(ACCOUNT_ID))
        require(args.amountMinor in 1..MAX_TRANSFER_MINOR)
        require(args.currency in ALLOWED_CURRENCIES)
    }
~~~

Parsing a string copy is acceptable only after a byte cap; a streaming decoder is preferable for larger read-only payloads. The checks still do not authorize the account, tenant, destination, or transfer. Authorization runs immediately before the effect and binds the canonical argument digest to the approval/policy decision.

## Schema evolution

Persist an event type and integer schema version. Readers should:

- read all versions still present in retention;
- upcast old events with pure deterministic transformations;
- never reinterpret an old field in place;
- add optional fields with defined defaults;
- introduce a new event/version for changed meaning;
- retain original payload digest for audit.

Tool schemas need a stable tool version. Changing required fields or semantics while keeping the same name/version can make replay or approval unsafe.

## Structured-output recovery

If model output fails parsing or validation:

1. retain the bounded raw response and validation diagnostics securely;
2. classify truncation, provider refusal, syntax error, and schema violation separately;
3. retry only within the run budget;
4. send concise machine-readable correction feedback;
5. cap repair attempts;
6. never execute a partially parsed tool call.

Provider “structured output” means the provider attempted to satisfy a supported schema subset. It does not establish provenance, tenant ownership, freshness, policy, or effect safety. Treat the returned object as untrusted input, run the same business validators as ordinary API input, and store the provider/model/schema version with the validation result. A successful decode followed by failed business validation is a model-result defect, not a transport retry.

## JSON Schema details

Declare the dialect. Draft 2020-12 changed tuple handling to <code>prefixItems</code>/<code>items</code> and formalized unevaluated keywords. Omitting <code>additionalProperties</code> permits additional properties, so executable tool schemas usually need explicit closure. Confirm the consumer implements the chosen vocabulary; provider structured-output subsets may reject valid full-dialect constructs.

## Checklist

- [ ] Maximum bytes, nesting, strings, arrays, and numbers are enforced.
- [ ] Dialect and supported subset are recorded.
- [ ] Unknown-field policy is boundary-specific.
- [ ] Polymorphism is allowlisted.
- [ ] Kotlin missing/null/default semantics are tested.
- [ ] Java/Kotlin generated schemas have golden compatibility tests.
- [ ] Persisted events have explicit versions and deterministic upcasters.
- [ ] Invalid or partial tool arguments never execute.

## Sources

- [JSON Schema Draft 2020-12](https://json-schema.org/draft/2020-12)
- [Kotlin JSON configuration](https://kotlinlang.org/docs/serialization-json-configuration.html)
- [Kotlin PolymorphicSerializer](https://kotlinlang.org/api/kotlinx.serialization/kotlinx-serialization-core/kotlinx.serialization/-polymorphic-serializer/)
- [Jackson project version status](https://github.com/FasterXML/jackson)
- [Jackson polymorphic deserialization security](https://github.com/FasterXML/jackson-docs/wiki/JacksonPolymorphicDeserialization)
- [OpenAI Java SDK structured outputs](https://github.com/openai/openai-java)
