# DanielMckinlay.com

Source code for my personal website, [danielmckinlay.com](https://danielmckinlay.com).

## Status

🚧 Under construction. A placeholder page is live while the full site is designed.

More coming. Be patient. Will add when I have time. 

## Stack

- **Site:** static HTML and CSS
- **Version control:** Git and GitHub
- **Hosting (planned):** Amazon S3 behind Amazon CloudFront
- **DNS:** Amazon Route 53, domain registered with NameCheap
- **TLS:** AWS Certificate Manager

## Project structure

```
.
├── site/            # Everything deployed to the web
│   ├── index.html
│   ├── 404.html
│   ├── robots.txt
│   └── css/
│       └── style.css
├── .editorconfig
├── .gitignore
├── LICENSE
└── README.md
```

## Run locally

No build step. Open `site/index.html` in a browser, or serve the folder:

```bash
cd site
python3 -m http.server 8000
```

Then visit http://localhost:8000.

## License

Code is MIT licensed. See [LICENSE](LICENSE). Written content and images are © Daniel McKinlay.
