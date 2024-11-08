# PrimeCodeGen

> Resolve specification issues and generate code from OpenAPI specifications.

This project is a collection of tools to help with merging, patching, and generating code (including user-provided templates) from API specifications.

- OpenAPI 3.0
- OpenAPI 3.1

## Installation

* Install Go on your system
* Clone this repo
* Change into the cloned repo directory
* Type ```$ go build``` and use compiled the `./primecodegen` binary inside the current directory
* Optionally type ```$ go install``` to use `primecodegen` as systemwide command

## OpenAPI Code Generator

The `openapi-generate` command can be used to generate code from an OpenAPI specification, using a built-in or custom template.

| Command                                                                 | Description                                                   |
| ----------------------------------------------------------------------- | ------------------------------------------------------------- |
| `primecodegen openapi-generate -i openapi.yaml -g go -t client -o /out` | run code generation with generator `go` and template `client` |

Environment Variables:

- `PRIMECODEGEN_DEBUG_SPEC` - if set, the final OpenAPI specification is written to stdout.
- `PRIMECODEGEN_DEBUG_TEMPLATEDATA` - if set, the template data passed to the code generator is written to stdout.

## OpenAPI Merger

The `openapi-merge` command can be used to convert an input specification into another format. 

| Command                                                                                 | Description                                                                                                                                                                          |
| --------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------ |
| `primecodegen openapi-merge --empty /in/empty_spec.yaml --input /in --output-dir /out ` | Merge OpenAPI specifications to be compatible with code generation tool. Provide an empty OpenAPI 3.0 spec to  build up a clean info-block. As an alternative use the built-in merge when using `openapi-patch` with multiple input specs. |

**Note**: If ` --empty` paremeter is not provided the first API spec is taken as starting point for the merged spec resulting in duplication of contents inside the infoblock.

Example for an empty OpenAPI 3.0 spec:
```yaml
openapi: "3.0.1"
info:
  title: ""
  version: ""
  summary: ""
  description: ""
  contact: ""
  extensions: ""
  license: ""
  termsOfService: ""
paths: {}
```

## OpenAPI Converter

The `openapi-convert` command can be used to convert an input specification into another format. 

| Command                                                                                                                                             | Description                                                                             |
| --------------------------------------------------------------------------------------------------------------------------------------------------- | --------------------------------------------------------------------------------------- |
| `primecodegen openapi-convert --converter-url https://host.converter  --format-in swagger20 --format-out openapi30 --input /in --output-dir /out  ` | Converts input - into output format (currently Swagger 2.0 to OpenAPI 3.0 is supported) |

**Note**: If ` --converter-url` paremeter is not provided and env var `PRIMECODEGN_SWAGGER_CONVERTER` is not set the default convert `https://converter.swagger.io/api/convert` will be used.

Environment Variables:

- `PRIMECODEGN_SWAGGER_CONVERTER` - if set, the respective converter for Swagger 2.0 to OpenAPI 3.0 conversion is used. The complete url must be provided, e.c. `https://converter.swagger.io/api/convert`

## OpenAPI Template Data

The `openapi-generate-template` command can be used to pre-process the openapi spec and pass the resulting template data to an external code generator.
The command supports the following options:

| Command                                                                                      | Description                            |
| -------------------------------------------------------------------------------------------- | -------------------------------------- |
| `primecodegen openapi-export-template-data -i openapi.yaml -g go -t client`                  | generate go template data, stdout      |
| `primecodegen openapi-export-template-data -i openapi.yaml -g go -t client -o template.yaml` | generate go template data, file output |

## OpenAPI Patch

The `openapi-patch` command can be used to apply automatic modifications, merge multiple specifications, and apply custom patches to the OpenAPI specification.

| Command                                                                            | Description                                                      |
| ---------------------------------------------------------------------------------- | ---------------------------------------------------------------- |
| `primecodegen openapi-patch -i openapi.yaml -o patched.yaml`                       | if no patches are specified, the default ones are applied        |
| `primecodegen openapi-patch -i openapi.yaml -i openapi.part2.yaml -o patched.yaml` | merge one or more specifications into one                        |
| `primecodegen openapi-patch -i openapi.yaml -p flattenSchemas -o patched.yaml`     | apply built-in patch with id `flattenSchemas`                    |
| `primecodegen openapi-patch -i openapi.yaml -f noservers.jsonpatch`                | apply a [jsonpatch](https://jsonpatch.com/) to the specification |
| `primecodegen openapi-patch -i openapi.yaml -f mypatch.patch`                      | apply a `git patch` to the specification                         |
| `primecodegen openapi-patch list`                                                  | list available patches                                           |

**Note**: All the options can be combined, e.g. merging multiple specifications, custom user-provided patches and built-in patches.

The following built-in patches are available:

| Patch                             | Default | Description                                                                                                                                                   |
| --------------------------------- | ------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| `pruneOperationTags`              | false    | Removes all tags from operations.                                                                                                                             |
| `pruneOperationTagsExceptFirst`   | false   | Removes all tags from operations except the first one.                                                                                                        |
| `pruneCommonOperationIdPrefix`    | false   | Removes common operation id prefixes (e. g. all operationIds start with `API_`)                                                                               |
| `generateOperationIds`            | true   | Generates operationIds for all operations based on the HTTP path and method, overwriting existing ones.                                                       |
| `flattenSchema`                   | false    | Flattens inline request bodies and response schemas into the components section of the document.                                                              |
| `missingSchemaTitle`              | true    | Adds a title to all schemas that are missing a title.                                                                                                         |
| `createOperationTagsFromDocTitle` | false   | Removes all tags and creates one new tag per API spec from the document title, setting it on each operation. This patch will be applied before merging specs. |
| `inlineAllOfHierarchies`          | false   | Inlines properties of allOf-referenced schemas and removes allOf-references in schemas                                                                        |

**Note**: The patches are applied in the order you specify them in. `createOperationTagsFromDocTitle` is an exception to that rule because it is always applied first before specs are possibly merged. If none are specified, the patches flagged as `default` are applied. 

## Roadmap

- [ ] Add support for AsyncAPI (https://github.com/asyncapi/parser-go/tree/master)
- [ ] Add support for Protobuf (https://github.com/yoheimuta/go-protoparser)

## Credits

- OpenAPI Parser: [libopenapi](https://github.com/pb33f/libopenapi)
- Patches - Git: [go-gitdiff](https://github.com/bluekeyes/go-gitdiff)
- Patches - JSON: [jsonpatch](https://github.com/evanphx/json-patch)

## License

Released under the [MIT license](./LICENSE).
