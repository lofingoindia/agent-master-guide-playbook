# Schemas, JSON, and Structured Output

> **Last researched:** 2026-08-31
> **Applies to:** .NET 10 LTS, C# 14

An agent crosses several contracts that are often collapsed into one: the C# type, the local serialization contract, JSON Schema, the provider's supported schema subset, and domain authorization. They must be versioned and validated independently.

## Contract layers

~~~mermaid
flowchart LR
    Type[C# DTO] --> Json[System.Text.Json contract]
    Json --> Schema[JSON Schema]
    Schema --> Provider[Provider subset]
    Provider --> Parse[Strict local parse]
    Parse --> Domain[Business validation]
    Domain --> Auth[Authorization and effect policy]
~~~

Provider-constrained decoding can guarantee schema conformance under documented conditions. It cannot prove that a customer ID exists, a path belongs to the tenant, a quantity is affordable, or the caller may perform the action.

## Strict local parsing

.NET 10 adds <code>JsonSerializerOptions.Strict</code>, a useful secure starting point. The preset disallows unmapped members and duplicate properties, uses case-sensitive matching, and respects nullable annotations and required constructor parameters.

~~~csharp
private static readonly JsonSerializerOptions ToolJson =
    new(JsonSerializerOptions.Strict)
    {
        MaxDepth = 32,
        TypeInfoResolver = ToolJsonContext.Default
    };
~~~

The ordinary serializer defaults are more permissive: unknown properties and duplicate names can be accepted. Do not assume a C# <code>required</code> property, nullable annotation, or constructor parameter has the same meaning under every options instance.

Before parsing:

- cap UTF-8 bytes;
- reject unsupported encodings and unexpected content types;
- cap nesting and collection sizes;
- avoid materializing an unlimited response as a string.

After parsing:

- validate ranges, formats, cross-field invariants, and tenant ownership;
- canonicalize paths, hosts, and identifiers before policy checks;
- reject extra semantic states even when syntactically valid;
- preserve a safe validation code rather than echoing sensitive input.

## Source generation and Native AOT

Use System.Text.Json source generation for known wire contracts. It reduces reflection, startup work, and trimming risk, and it is the supported direction for Native AOT. Set <code>JsonSerializerIsReflectionEnabledByDefault</code> to false in AOT-oriented applications so an accidental reflection path fails during testing rather than in production.

Source generation has two modes: metadata and fast-path serialization. Feature support differs. Test both serialization and deserialization for every contract used by tools, state, queues, MCP, and providers.

## Generating JSON Schema

<code>JsonSchemaExporter</code>, introduced in .NET 9, can derive a schema from the configured System.Text.Json contract. This reduces drift, but the generated schema is not automatically the right public tool contract.

Apply an explicit transformation step to:

- add stable tool and property descriptions;
- set <code>additionalProperties: false</code> where required;
- represent nullable versus optional fields correctly;
- remove unsupported keywords for the chosen provider;
- simplify deep unions and polymorphism;
- assign a schema ID/version and golden-file test it.

Keep the canonical domain schema provider-neutral. Generate or transform provider-specific variants and validate fixtures against each.

## Provider structured output

OpenAI Structured Outputs use JSON Schema response formats or strict function tools on supported models. Anthropic provides JSON output schemas and <code>strict: true</code> tool use, with documented platform/model availability and schema complexity limits. These capabilities evolve independently, so negotiate against the selected model and endpoint rather than assuming the SDK package implies support.

Always handle non-success terminal states separately:

- refusal or policy stop;
- output-length truncation;
- canceled or incomplete response;
- stream failure after partial arguments;
- provider-side schema compilation failure;
- unsupported schema keyword or model.

Never execute a tool from partial streamed JSON. Wait for the provider's completed tool-call event, then parse and validate locally.

## Versioning

Additive schema changes can still change model behavior or break strict <code>additionalProperties</code> consumers. Use stable tool names or an explicit version field. Persist the schema/tool version with each invocation and effect record so replay uses compatible decoding.

For durable state, write upcasters or migrations. Do not deserialize historical payloads directly into the newest type and hope defaults preserve meaning.

## Failure patterns

- Parsing free-form prose with a regex when the decision should be a structured field.
- Trusting provider schema validation as authorization.
- Accepting unknown fields that alter a downstream dynamic object.
- Conflating missing, null, empty, and default values.
- Generating a schema from a DTO and publishing it without provider-subset tests.
- Reflection-only serialization succeeds locally but fails after trimming/AOT.
- Executing a partially streamed tool call.

## Review checklist

- [ ] Input bytes, depth, strings, and collections are bounded.
- [ ] Unknown and duplicate properties have an explicit policy.
- [ ] Provider output is parsed and domain-validated locally.
- [ ] Tool calls execute only after a terminal completed event.
- [ ] Schema versions are stored with durable events and effects.
- [ ] Source-generation and AOT paths are tested from published output.
- [ ] Provider schema subsets have golden compatibility tests.

## Primary sources

- [.NET 10 System.Text.Json changes](https://learn.microsoft.com/en-us/dotnet/core/whats-new/dotnet-10/libraries)
- [System.Text.Json source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/source-generation)
- [Reflection versus source generation](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/reflection-vs-source-generation)
- [Handle unmapped JSON members](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/missing-members)
- [Required properties](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/required-properties)
- [Extract JSON Schema from .NET types](https://learn.microsoft.com/en-us/dotnet/standard/serialization/system-text-json/extract-schema)
- [OpenAI Responses API structured JSON](https://developers.openai.com/api/reference/resources/responses/methods/create)
- [Anthropic structured outputs](https://platform.claude.com/docs/en/build-with-claude/structured-outputs)
