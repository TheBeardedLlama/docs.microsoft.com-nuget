---
# required metadata 

title: Package Content | NuGet V3 API | Microsoft Docs
author:
- joelverhagen
- kraigb
ms.author:
- joelverhagen
- kraigb
manager: skofman
ms.date: 10/26/2017
ms.topic: reference
ms.prod: nuget
ms.technology: null
ms.assetid: ec68b5d1-a684-4995-b1a6-6210dbb24875

# optional metadata

description: The package base address is a simple interface for fetching the package itself.
keywords: flat container, dnx, package base address, nupkg, versions, unlisted, enumerate, download
ms.reviewer:
- karann
- unniravindranathan

---

# Package Content

It is possible to generate a URL to fetch an arbitrary package's content (the .nupkg file) using the V3 API. The
resource used for fetching package content is the `PackageBaseAddress` resource found in the
[service index](service-index.md). This resource also enables discovery all versions of a package, listed or unlisted.

This resource is commonly referred to as either the "package base address" or as the "flat container".

## Versioning

There is one `@type` value for the package base address: `PackageBaseAddress/3.0.0`.

## Base URL

The base URL for the following APIs is the value of the `@id` property associated with the aforementioned
resource `@type` value. In the following document, this placeholder URL will be used:

```
https://example/flatcontainer/
```

## HTTP methods

All URLs found in the package base resource only support the HTTP method `GET`.

## Enumerate package versions

If the client knows a package ID and wants to discover which package versions the V3 package source has available, the
client can construct a predictable URL to enumerate all package versions. This list is meant to be a "directory
listing" for the package content API mentioned below.

For example, this is the version list returned by nuget.org when enumerating the versions of `owin` in the flat
container.

[!code-JSON [package-base-address-index.json](./_data/package-base-address-index.json)]

This list contains both listed and unlisted package versions.

```
GET https://example/flatcontainer/{LOWER_ID}/index.json
```

### Request parameters

Name     | In     | Type    | Notes
-------- | ------ | ------- | -----
LOWER_ID | URL    | string  | Required: the package ID, lowercase

The `LOWER_ID` value is the desired package ID lowercased using the rules implemented by .NET's
[`System.String.ToLowerInvariant()`](https://msdn.microsoft.com/en-us/library/system.string.tolowerinvariant.aspx)
method.

### Response

If the package source has no versions of the provided package ID, a 404 status code is returned.

If the package source has one or more versions, a 200 status code is returned. The response body is a JSON object with
the following property:

Name     | Type             | Notes
-------- | ---------------- | -----
versions | array of strings | Required: the package IDs available

The strings in the `versions` array are all lowercased, normalized NuGet version strings.

## Download package content (.nupkg)

If the client knows a package ID and version and wants to download the package content, they only need construct the
following URL:

```
GET https://example/flatcontainer/{LOWER_ID}/{LOWER_VERSION}/{LOWER_ID}.{LOWER_VERSION}.nupkg
```

### Request parameters

Name          | In     | Type    | Notes
------------- | ------ | ------- | -----
LOWER_ID      | URL    | string  | Required: the package ID, lowercase
LOWER_VERSION | URL    | integer | Required: the package version, normalized and lowercase

Both `LOWER_ID` and `LOWER_VERSION` are lowercased using the rules implemented by .NET's
[`System.String.ToLowerInvariant()`](https://msdn.microsoft.com/en-us/library/system.string.tolowerinvariant.aspx)
method.

The `LOWER_VERSION` is the desired package version normalized using NuGet's version
[normalization rules](../../reference/package-versioning.md#normalized-version-numbers). This means that build metadata
that is allowed by the SemVer 2.0.0 specification must be excluded in this case.

### Response body

If the package exists on the package source, a 200 status code is returned. The response body will be the package
content itself.

If the package does not exists, a 404 status code is returned.