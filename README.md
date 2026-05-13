# cloudinary-shopify-theme-delivery

Code examples for using Cloudinary to deliver assets in Shopify Liquid themes.

This repository contains two related but different Liquid approaches:

- `assetlink/`: for stores that use the Cloudinary AssetLink app to sync or upload assets to Shopify.
- `delivery-only/`: for stores that mirror Shopify assets into Cloudinary through upload mapping, without AssetLink metadata or AssetLink filename conventions.

## When to use `assetlink/`

Use the snippets under `assetlink/` when your Shopify product images or Shopify Files are created by the Cloudinary AssetLink app.

AssetLink can write enough Cloudinary information into Shopify filenames for the theme to reconstruct the original Cloudinary delivery URL. These files use an encoded filename pattern like:

```text
cloudinary__<cloud_name>__image__upload__...__<public_id>__...__cld.<extension>
```

The `assetlink/` snippets understand that filename format and convert matching Shopify CDN URLs back into Cloudinary URLs at render time. This lets a Shopify theme display images through Cloudinary while still using Shopify product media and Shopify Files that were synced by AssetLink.

Use this mode when:

- Product media was linked or uploaded through the Cloudinary AssetLink app.
- You see Shopify media filenames that start with `cloudinary__`.
- You want the theme to use the exact Cloudinary asset identity encoded by AssetLink.
- You need AssetLink product media and swatch workflows to coexist with Cloudinary delivery.

The key snippets are:

- `assetlink/snippets/cloudinary-assetlink-url.liquid`: Converts Shopify CDN URLs whose filenames follow the AssetLink `cloudinary__...` convention.
- `assetlink/snippets/cloudinary-synced-url.liquid`: Converts normal Shopify CDN URLs by mapping the Shopify CDN path into a synced Cloudinary folder. It intentionally skips filenames that look like AssetLink files.

In practice, use `cloudinary-assetlink-url` for AssetLink-generated filenames, and use `cloudinary-synced-url` only for product media that is mirrored to Cloudinary by path/folder sync rather than by AssetLink filename encoding.

## Working with the AssetLink snippets

1. Copy the relevant snippets from `assetlink/snippets/` into your Shopify theme's `snippets/` directory.
2. Copy or merge the settings from `assetlink/config/settings_schema.json` into your theme settings.
3. Configure the Cloudinary settings in the Shopify theme editor, including the Cloudinary hostname and default transformations.
4. Wrap rendered HTML that contains Shopify media URLs with the snippet.

Typical usage pattern:

```liquid
{% capture media_html %}
  {{ product.featured_media | image_url: width: 1200 | image_tag }}
{% endcapture %}

{% render 'cloudinary-assetlink-url', content: media_html %}
```

If the content may contain both AssetLink filenames and folder-synced Shopify CDN paths, run the AssetLink-aware snippet first, then the synced-folder snippet:

```liquid
{% capture media_html %}
  {{ product.media | where: 'media_type', 'image' | first | image_url: width: 1200 | image_tag }}
{% endcapture %}

{% capture assetlink_html %}
  {% render 'cloudinary-assetlink-url', content: media_html %}
{% endcapture %}

{% render 'cloudinary-synced-url', content: assetlink_html %}
```

This order avoids treating AssetLink-encoded filenames as normal folder-synced assets.

## When to use `delivery-only/`

Use `delivery-only/` when you are not using the AssetLink app for the images rendered by the theme.

This approach assumes Shopify images are mirrored to Cloudinary through upload mapping or another folder-based sync process. The snippet converts Shopify CDN paths into Cloudinary paths using configured prefixes and transformations. It does not read AssetLink filename metadata, product media metadata, or swatch metadata.

Use this mode when:

- Images were uploaded directly to Shopify and mirrored to Cloudinary by upload mapping.
- There is no AssetLink sync involved for the rendered images.
- Shopify filenames do not contain the AssetLink `cloudinary__...` pattern.
- You only need path-based Shopify CDN to Cloudinary URL conversion.

The main snippet is:

- `delivery-only/snippets/cloudinary-url.liquid`: Converts Shopify CDN image and video URLs to Cloudinary URLs based on theme settings.

Typical usage pattern:

```liquid
{% capture media_html %}
  {{ product.featured_media | image_url: width: 1200 | image_tag }}
{% endcapture %}

{% render 'cloudinary-url', content: media_html %}
```

## Choosing the right mode

Use `assetlink/` for media managed by the Cloudinary AssetLink app. Use `delivery-only/` for a plain Shopify-to-Cloudinary delivery setup where upload mapping mirrors assets and the theme maps Shopify CDN URLs by path.

Do not use `delivery-only/snippets/cloudinary-url.liquid` as the primary path for AssetLink-synced images, because it cannot decode the AssetLink filename convention. Conversely, do not require the AssetLink snippets for stores that only use upload mapping and never sync media with AssetLink.
