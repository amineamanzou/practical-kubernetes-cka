# Practical Kubernetes CKA Journey

This repository documents my learning journey towards achieving the Kubernetes Administrator Certification (CKA). It utilizes the Zettelkasten method to create atomic, interconnected notes on various Kubernetes concepts and practices.

## About This Project

The goal of this project is to build a comprehensive knowledge base for the CKA exam, structured using the Zettelkasten approach. This method emphasizes:

- **Atomic Notes**: Each concept gets its own page.
- **Linking**: Notes are connected to build a network of knowledge.
- **Organic Growth**: The documentation evolves as learning progresses.

## Running Locally

To view or contribute to this documentation locally, you need to have [Hugo](https://gohugo.io/installation/) installed.

1. **Clone the repository:**

    ```bash
    git clone https://github.com/your-username/practical-kubernetes-cka.git # Replace with your repo URL
    cd practical-kubernetes-cka
    ```

2. **Initialize Submodules (if using a theme as a submodule):**
    If the Hextra theme is included as a Git submodule (check for a `.gitmodules` file), initialize it:

    ```bash
    git submodule update --init --recursive
    ```

    *Note: If the theme is added differently, this step might not be needed.*

3. **Run the Hugo server:**

    ```bash
    hugo server
    ```

    This command builds the site and starts a local server, usually accessible at `http://localhost:1313/`. The site will automatically rebuild when you make changes to the content.

## Updating Your Fork / Contributing

If you have forked this repository and wish to update it or contribute back (if applicable):

1. **Fork the repository** on GitHub.
2. **Clone your fork** locally (`git clone https://github.com/your-fork-username/practical-kubernetes-cka.git`).
3. **Make your changes** within the `content/` directory, following the established structure and formatting guidelines (see `CONTRIBUTING.md` if one exists, or refer to the project's documentation standards).
4. **Preview your changes** locally using `hugo server`.
5. **Commit and push** your changes to your fork.
6. **(Optional) Open a Pull Request** to the original repository if you wish to contribute your changes back.

## Project Structure

- **Content**: All markdown notes reside in the `content/` directory. Follow the naming conventions (`kebab-case.md`) and Zettelkasten principles.
- **Images**: Store images in `content/images/`.
- **Configuration**: Hugo site configuration is in `hugo.toml` (or `config.toml`/`config.yaml`).
- **Theme**: This project uses the [Hextra](https://imfing.github.io/hextra/docs/) theme.

Feel free to explore the notes and follow the connections!
