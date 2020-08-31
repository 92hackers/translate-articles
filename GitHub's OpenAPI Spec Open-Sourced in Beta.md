# GitHub's OpenAPI Spec Open-Sourced in Beta

GitHub has [open sourced an OpenAPI description of its API](https://github.blog/2020-07-27-introducing-githubs-openapi-description/). Aimed to allow developers to discover the API capabilities first-hand, [GitHub's OpenAPI](https://github.com/github/rest-api-description) also enables the programmatic creation of mock servers, test suites, and language bindings.

According to GitHub, its OpenAPI description contains more than 600 operations that can be explored using a [Postman Collection](https://www.postman.com/collection/). Two formats are provided: a **bundled** version which is easier to read and use, and a **dereferenced** version for tools that have poor support for inline references to components. The bundled version is based on OpenAPI Components, which use the [JSON Reference specification](https://tools.ietf.org/html/draft-pbryan-zyp-json-ref-03) to bundle together multiple files into a unique, coherent entity, which makes reuse and collaboration easier.

> Describing a 12-year-old API is no easy task. We’ve built this description using a mix of existing JSON schemas, documented examples, contract testing, and love. We expect to make the description even more complete and accurate as we go forward and as OpenAPI becomes central to our developer experience — internally and externally.

GitHub used a few custom extensions to the standard to model concepts that are not a good fit with OpenAPI components. Such extensions are used to handle additional data, including a [displayName](https://github.com/github/rest-api-description/blob/main/extensions.md#x-displayname), and [custom information required by OctoKit](https://github.com/github/rest-api-description/blob/main/extensions.md#x-github).

Once you have imported GitHub's OpenAPI into [Postman](https://www.postman.com), you can easily browse all available operations and send request to GitHub REST API, as the image below shows for a specific case:

![](https://static.breword.com/infoq-github-openapi.jpeg)

The [OpenAPI Specification](https://www.openapis.org) (OAS) defines a standard, programming language-agnostic way of describing HTTP APIs. It serves both as a contract between API provider and API consumer, as well as single source of truth. One of the advantages granted by an OpenAPI spec is enabling front-end and back-end developers to work independently by relying on an automatically generate API mockup.

GitHub plans to make releases available on a quarterly basis for GitHub Enterprise Server and GitHub Private Instances, and more frequently for the public facing github.com API.


