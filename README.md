# yokozuna-dohyoiri-lineage
SPA visualising the lineage of [yokozuna](https://en.wikipedia.org/wiki/Makuuchi#Yokozuna) ring entering ceremonies.

## Contributing
### Data corrections/expansions
Refer to `src/assets/data.json` as this file contains the data driving the app.

### Code
The app is built with the [Svelte](https://svelte.dev/) (no Kit) component framework.

The graph is rendered as a [d3-hierarchy/tree](https://d3js.org/d3-hierarchy/tree).   

## Building/running locally
### Prerequisites
- Node.js or whichever runtime you prefer
- npm

Grab dependencies:
```shell
npm install
```
Run locally:
```shell
npm run dev
```
Package to static files:
```shell
npm run build
```

## Deployment
A [GitHub Actions workflow](https://github.com/bleloch/yokozuna-dohyoiri-lineage/blob/main/.github/workflows/build.yml) pushes changes [here](https://bleloch.github.io/yokozuna-dohyoiri-lineage/) upon merge to `main`.

## Credits
- [Hankegami's fine research on the subject](), which inspired me to build this.
- [Wikipedia - List of yokozuna](https://en.wikipedia.org/wiki/List_of_yokozuna)
- [SumoDB](http://sumodb.sumogames.de/Default.aspx)