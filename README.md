# wiki.habibullah.dev

> **My Digital Garden & Second Brain.**
> Hosted specifically at [wiki.habibullah.dev](https://wiki.habibullah.dev)

This repository contains the source code and content for my personal wiki. It serves as a central knowledge base for my development notes, Linux configurations, CTF writeups, and other technical documentation.

## Tech Stack

- **Engine:** [Habibullah](https://wiki.habibullah.dev/) (Static Site Generator)
- **Hosting:** Cloudflare Pages
- **DNS:** Cloudflare
- **Editing:** Obsidian (locally)

## 🛠️ Local Development

To run this project locally, ensure you have **Node.js v22.xx** installed.

1.  **Clone the repository:**

    ```bash
    git clone https://github.com/md8-habibullah/wiki-habibullah.git
    ```

    ```bash
    cd wiki-habibullah
    ```

2.  **Install dependencies:**
    This project uses `pnpm` for package management.

    ```bash
    pnpm install
    ```

3.  **Run the local server:**
    This will build the site and start a hot-reloading preview server at `http://localhost:8080`.
    ```bash
    pnpm quartz build --serve
    ```
    Alternatively use `npm`,
    ```bash
    npx quartz build --serve
    ```

    ## Deployment

    Deployment is automated via **Cloudflare Pages**.
    Any push to the `main` branch triggers a build and deployment to the edge network.

    * **Build Command:** `npx quartz build`
    * **Output Directory:** `public`

    ## Contact & Issues

    If you find an error in the documentation or have a suggestion:

    * **Email:** [wiki@habibullah.dev](mailto:wiki@habibullah.dev)
    * Or open an Issue in this repository.
