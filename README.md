# CapabilityInteractionGraphs
The official repository, tracking Capability Interaction Graph (CIG) versions.

## Versioning

CIG releases follow [Semantic Versioning 2.0.0](https://semver.org/). Each version number has the form `MAJOR.MINOR.PATCH`:

- **MAJOR** is incremented for backwards-incompatible changes, e.g. removing or renaming a class or property, or changing its meaning so that existing knowledge graphs or SPARQL queries no longer conform or behave as before.
- **MINOR** is incremented for backwards-compatible additions, e.g. new classes, properties or SHACL shapes that do not invalidate existing data.
- **PATCH** is incremented for backwards-compatible fixes that do not change the vocabulary, e.g. corrected labels, comments, typos or documentation.

Every release lives in its own top-level directory named after its version (e.g. [1.0.0/](1.0.0/)), containing:

- `capability-interaction-graph_<version>.ttl`: the ontology.
- `capability-interaction-graph-shapes.ttl`: the companion SHACL shapes.
- `web/`: the documentation web page for that version.

The ontology's `owl:versionIRI` must match the directory name (e.g. `http://purl.org/capability-interaction-graph/1.0.0`). Released versions are not modified; any change goes into a new version directory.

## Generating the web page

The base of the documentation page in `web/` is generated with [Widoco](https://github.com/dgarijo/Widoco). Download the latest `widoco-<x.y.z>-jar-with-dependencies.jar` from the [Widoco releases](https://github.com/dgarijo/Widoco/releases) (requires Java), then run from the version directory:

```sh
java -jar widoco-<x.y.z>-jar-with-dependencies.jar \
  -ontFile capability-interaction-graph_<version>.ttl \
  -outFolder web \
  -getOntologyMetadata \
  -uniteSections \
  -webVowl \
  -oops \
  -rewriteAll \
  -lang en
```

This produces `web/index.html` (a single page with all sections), the serializations (`.ttl`, `.owl`, `.nt`, `.jsonld`), the WebVOWL visualization (`webvowl/`), the OOPS! evaluation (`OOPSevaluation/`), the provenance page (`provenance/`) and the shared assets (`resources/`).

The page metadata (title, authors, release date, citation, license, abstract, etc.) is taken from the ontology's annotations, so update those in the `.ttl` file rather than in the generated HTML wherever possible.

> **Note:** re-running Widoco with `-rewriteAll` overwrites `index.html` and therefore removes the manual additions listed below. Re-apply them afterwards (the easiest way is to diff against the previous version's `web/index.html`).

## Manual additions to the web page

After generating the base with Widoco, the following was added by hand:

**Files**

- `web/capability-interaction-graph_<version>.ttl`: a copy of the versioned ontology file.
- `web/capability-interaction-graph-shapes.ttl` and `web/shacl/capability-interaction-graph-shapes.ttl`: copies of the SHACL shapes, the latter being the one linked from the page.

**`web/index.html`**

- **YASGUI query editor assets** in `<head>`: the stylesheet and script of [`@triply/yasgui`](https://www.npmjs.com/package/@triply/yasgui) loaded from unpkg.
- **CIGAR Studio badge** under *Visualization*, next to the WebVOWL badge, linking to [CIGAR Studio](https://cigar.manzi-it.com).
- **Validation** entry in the metadata list, with two badges:
  - *Validate with SHACL*, linking to `shacl/capability-interaction-graph-shapes.ttl`.
  - *Try in SHACL Playground*, linking to a pre-loaded [Zazuko SHACL Playground](https://s.zazuko.com/3tGoR4F).
- **Example section** (`<div id="Example">`), placed between the Overview and Description sections. It embeds a YASGUI SPARQL query editor, pre-filled with a default query listing capabilities with their bearers and consumed/produced resources, and run against the demo endpoint `https://sparql.manzi-it.com/demo/sparql`, which hosts an example CIG instance graph of the [Lycoming O-320](https://en.wikipedia.org/wiki/Lycoming_O-320) ignition system. YASGUI is initialised with `autofocus: false` so the page does not scroll down to the editor on load.
- **SHACL paragraph** at the end of the Description section, pointing to the shapes file and explaining what it validates beyond OWL 2 DL.

## GitHub Pages

The documentation of the **latest** version is published with GitHub Pages from the `gh-pages` branch, at <https://eppemedia.github.io/CapabilityInteractionGraphs/>.

The `gh-pages` branch does not follow the layout of `main`: its root is a copy of `<version>/web/` of the latest release (plus `LICENSE`), so that `index.html` and the serializations are served directly from the site root. Never edit `gh-pages` by hand; make changes in `main` and publish them.

To publish a new version, once its `<version>/web/` directory (including the [manual additions](#manual-additions-to-the-web-page)) is committed on `main`:

```sh
# Check out gh-pages next to the main working tree
git worktree add ../cig-gh-pages gh-pages
cd ../cig-gh-pages

# Replace the site contents with the new version's web page, keeping LICENSE
rsync -a --delete --exclude .git --exclude LICENSE ../CapabilityInteractionGraphs/<version>/web/ ./

git add -A
git commit -m "Add documentation of version <version> to github pages."
git push origin gh-pages

# Clean up
cd -
git worktree remove ../cig-gh-pages
```

Adjust `../CapabilityInteractionGraphs` to the path of your `main` checkout. GitHub Pages should be configured (under *Settings → Pages*) to deploy from the `gh-pages` branch, root folder. Older versions are not on the site, but they remain available in their version directories on `main`.
