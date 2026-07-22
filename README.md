# website

Visit at [abhimanyusikarwar.com](https://abhimanyusikarwar.com)

---

## Stack

- Framework: [Zola](https://www.getzola.org/)
- Theme: Adapted [zola-bearblog](https://codeberg.org/alanpearce/zola-bearblog) with some custom styles.
- Deployment: Cloudflare Pages

## Configuration

The site configuration is managed through `config.toml`. Here's how to modify different aspects:

1. Basic Settings:
   - Update `base_url` for your domain
   - Modify `title` and `description` for site metadata
   - Toggle features like `compile_sass`, `minify_html`, and `build_search_index`

2. Navigation:
   - Edit `[[extra.main_menu]]` entries to update navigation links
   - Format:
     ```toml
     [[extra.main_menu]]
     name = "Menu Item"
     url = "@/path/to/content.md"
     ```

3. Content Organization:
   - Posts go in the `content/posts` directory
   - TILs (Today I Learned) go in `content/tils`
   - Projects are configured in `content/projects/data.toml`

## Theme Customization

The site supports both light and dark themes through CSS variables in `static/styles/main.css`. Here's how to customize the colors:

### Base Theme Variables
```css
:root {
    /* Base styles */
    --font-primary: 'Poppins', sans-serif;
    --color-primary-text: #444;
    --color-background: #fff;
    --color-headline: #222;
    --color-link: #3273dc;
    --color-link-hover: #3273dc;
    --color-selection: #FDD9B5;
    --color-selection-text: #333;
    
    /* Tags */
    --color-tag-bg: #FFE4C7;
    --color-tag-border: #FFCBA4;
    --color-tag-text: #D35400;
    --color-tag-hover: #FFD9B0;
    
    /* Dark mode */
    --color-background-dark: #333;
    --color-text-dark: #ddd;
    --color-headline-dark: #eee;
    --color-link-dark: #8cc2dd;
    --color-code-bg-dark: #777;
    --color-blockquote-dark: #ccc;
    --color-input-bg-dark: #252525;
    --color-helptext-dark: #aaa;
    
    /* Other elements */
    --color-blogpost-visited: #8b6fcb;
}
```

### Color Usage

1. **Light Theme Colors**:
   - Background: `--color-background`
   - Text: `--color-primary-text`
   - Headlines: `--color-headline`
   - Links: `--color-link`
   - Tags: `--color-tag-bg`, `--color-tag-border`, `--color-tag-text`

2. **Dark Theme Colors**:
   - Background: `--color-background-dark`
   - Text: `--color-text-dark`
   - Headlines: `--color-headline-dark`
   - Links: `--color-link-dark`
   - Code blocks: `--color-code-bg-dark`
   - Form inputs: `--color-input-bg-dark`

3. **Special Elements**:
   - Text Selection: `--color-selection`, `--color-selection-text`
   - Visited Blog Posts: `--color-blogpost-visited`
   - Tag Hover: `--color-tag-hover`

### Theme Transition
The theme includes smooth transitions between light and dark modes:
```css
.theme-transition {
    transition: background-color 0.5s ease-in-out, 
                color 0.5s ease-in-out, 
                opacity 0.4s ease-in-out;
    opacity: 0.9;
}
```

## Deployment on Cloudflare Pages

### Setup Instructions

1. **Initial Setup**:
   - Sign in to your Cloudflare account
   - Navigate to "Pages" in the right navigation
   - Click "Create a project"
   - Select and connect your GitHub repository

2. **Project Configuration**:
   - Enter your project name (will be used for `{projectname}.pages.dev`)
   - Select your production branch
   - Choose "Zola" as the Framework preset

3. **Build Settings**:
   - Choose "Zola" as the Framework preset, which fills in the build command
     (`zola build`) and output directory (`public`)
   - Build image version: v3, which ships Zola 0.22.1
   - Add environment variable:
     - Name: `ZOLA_VERSION`
     - Value: `0.22.1`

4. Click "Save and Deploy"

Note that this site needs Zola 0.19 or newer, because `config.toml` uses
`generate_feeds` and `feed_filenames`. Those keys were named in the singular
before 0.19.

### Preview deployments

Preview builds are served from a branch URL such as
`https://branch-name.project.pages.dev`, but `base_url` in `config.toml` is
hardcoded to the production domain. Since Zola writes absolute URLs into
canonical tags, Open Graph tags, feeds, and the sitemap, a preview built with
the production `base_url` points all of them at the live site.

Replace the build command with this to build previews against their own URL:

```sh
if [ "$CF_PAGES_BRANCH" = "main" ]; then zola build; else zola build --base-url $CF_PAGES_URL; fi
```

`CF_PAGES_BRANCH` and `CF_PAGES_URL` are supplied by Cloudflare Pages.

### Troubleshooting

If the build fails with `zola: not found`, the build image is too old to include
Zola. Set the build image to v3 rather than installing Zola through a pre-build
script.

If the build fails inside a `UNSTABLE_PRE_BUILD` environment variable, delete
that variable. Older setups used it to install Zola through asdf, which breaks
on any build after the first: Cloudflare caches the asdf plugin directory, so
`asdf plugin add` exits with code 2 and takes the rest of the command with it.

For more details, check the [Cloudflare Pages documentation](https://developers.cloudflare.com/pages/) and [Zola deployment guide](https://www.getzola.org/documentation/deployment/cloudflare-pages/).