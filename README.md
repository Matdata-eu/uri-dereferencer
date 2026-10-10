# URI Dereferencer

Lightweight nginx Docker container for dereferencing Linked Data URIs via SPARQL DESCRIBE queries. Renders an interactive HTML view with property tables, WKT geometry on Leaflet maps, and content negotiation for RDF formats (Turtle, RDF/XML, JSON-LD). All JS libraries are bundled locally — no CDN tracking.

## Quick Start

Pull the pre-built image from GitHub Container Registry:

```bash
docker pull ghcr.io/matdata-eu/uri-dereferencer:latest
docker run -p 8080:8080 \
  -e SPARQL_ENDPOINT=https://example.org/sparql \
  -e BASE_URI=https://example.org \
  ghcr.io/matdata-eu/uri-dereferencer:latest
```

Or build locally:

```bash
docker-compose up -d
# or
docker build -t uri-dereferencer .
docker run -p 8080:8080 -e SPARQL_ENDPOINT=https://jena.matdata.eu/rinf/sparql uri-dereferencer
```

Then open `http://localhost:8080/{resource-uri}`.

### Configuration

| Variable | Default | Description |
|---|---|---|
| `SPARQL_ENDPOINT` | `https://jena.matdata.eu/rinf/sparql` | SPARQL endpoint URL |
| `BASE_URI` | `https://data.matdata.eu` | Entity namespace for local vs external URI routing and nginx content negotiation |

## Usage

**Production** — the URL *is* the URI:
```
https://data.matdata.eu/_netPointReferences_swi366_on_ne_348
```

**Local testing** — pass the full URI in the path:
```
http://localhost:8080/http://example.org/resource/123
```

### Content Negotiation

Click the format badges in the UI, or use `Accept` headers:

```bash
curl -H "Accept: text/turtle" http://localhost:8080/{uri}
curl -H "Accept: application/rdf+xml" http://localhost:8080/{uri}
curl -H "Accept: application/ld+json" http://localhost:8080/{uri}
```

### Geospatial

WKT geometries linked via `gsp:hasGeometry` are automatically rendered on a Leaflet map. Supports standard WKT, EWKT with SRID, and OpenGIS CRS format. Built-in EPSG definitions for 4326, 3857, 31370, 4258, 3035, 25832, 25833; others fetched from epsg.io.

## Architecture

nginx serves a client-side SPA. The browser extracts the URI from the URL path, runs `DESCRIBE` against the SPARQL endpoint, parses N-Triples with N3.js, and renders properties, geometry (betterknown + Leaflet + proj4), and related resources.

```
├── Dockerfile / docker-compose.yml / nginx.conf / entrypoint.sh
├── index.html
├── assets/
│   ├── js/          (config.js, dereferencer.js, map-viewer.js)
│   ├── css/         (dereferencer.css)
│   ├── data/        (prefixes.json)
│   └── vendor/      (bootstrap, bootstrap-icons)
└── vendor/          (n3, leaflet, proj4, betterknown, yasgui-graph-plugin)
```

## Dependencies

All libraries bundled locally:

| Library | Version | License | Purpose |
|---|---|---|---|
| [N3.js](https://github.com/rdfjs/N3.js) | 1.17.2 | MIT | RDF parsing |
| [Leaflet](https://leafletjs.com) | 1.9.4 | BSD-2-Clause | Interactive maps |
| [proj4js](https://github.com/proj4js/proj4js) | 2.9.2 | MIT | Coordinate transforms |
| [betterknown](https://github.com/placemark/betterknown) | 1.1.1 | MIT | WKT → GeoJSON |
| [@matdata/yasgui-graph-plugin](https://github.com/Matdata-eu/yasgui-graph-plugin) | 1.6.1 | Apache-2.0 | Interactive graph (bundles vis-network) |
| [Bootstrap](https://getbootstrap.com/) | 5.3.3 | MIT | UI framework |
| [Bootstrap Icons](https://icons.getbootstrap.com/) | 1.11.3 | MIT | Icons |

[THIRD_PARTY_NOTICES.md](THIRD_PARTY_NOTICES.md) holds their copyright notices
and license texts. The page footer links to it (served at
`/assets/THIRD_PARTY_NOTICES.md`). Update it when you add or upgrade a library.

## Development

```bash
# Without Docker — any HTTP server works:
python -m http.server 8080
# Edit assets/js/config.js to set SPARQL_ENDPOINT, then open http://localhost:8080/{uri}

# Rebuild Docker:
docker-compose down && docker-compose build --no-cache && docker-compose up -d
```

## Questions, bugs and feature requests

Please open an issue on GitHub:
<https://github.com/Matdata-eu/uri-dereferencer/issues>.

## Credits

URI Dereferencer is written by Mathias Vanden Auweele ([Matdata](https://matdata.eu/)).
It runs on [nginx](https://nginx.org/) and uses the open source libraries listed
under [Dependencies](#dependencies). Map tiles come from
[OpenStreetMap](https://www.openstreetmap.org/copyright), © OpenStreetMap contributors.

## License

Licensed under the [Apache License 2.0](LICENSE). Part of the [Matdata](https://matdata.eu/) semantic-tools collection.
