# wasm_toolchain_wsl
🧰 **A workaround toolchain for converting C/C++ code and libraries into wasm for web development with compiled js scripts.**

## Description

A toolchain targeted for **Windows** users navigating through system using **WSL2** (Windows Subsystem for Linux). This toolchain helps convert native c++ code and libraries
into **wasm** (Web Assembly) using **Emscripten** and **autoconf automake libtool**.

---

## Getting Started

### 🛠 Prerequisites

* Running Windows
* Installed WSL2 (latest version)

  * Installed Unix-based OS (This toolchain's commands refer to APT-based unix os like Ubuntu/Debian/Kali):
    
    ```
    wsl --install -d kali-linux 
    ```
* Installed VSCode (very lightweight) with C++ extension and WSL2 extension (for streamlining and debugging purpose)
  * Since the file systems for Windows and Linux are separated in a WSL2 environment, we streamline the workflow by running VSCode directly within the WSL2 Linux context. This ensures consistent path resolution       (especially in **c_cpp_properties.json**) and avoids cross-platform confusion.
    To do this, navigate to your project directory inside the WSL terminal and launch VSCode in full Linux mode:
    
    ```
    cd /home/user_name/project_dir
    ```
    Then, execute:
    ```
    code .
    ```
    This should open the targeted VSCode project in full linux mode
    
    
* Install additional dependencies, including full package **autoconf automake libtool** for portability and automated configuration:

  ```
  sudo apt install build-essential cmake git autoconf automake libtool
  ```
* Install **Emscripten** in the <user> directory of your linux <user> directory, this can be done via git clone:

  ```
  cd ~  (root <user> dir)
  git clone https://github.com/emscripten-core/emsdk.git
  cd emsdk  (new dir to store emcripten kit)
  ./emsdk install latest
  ./emsdk activate latest
  source ./emsdk_env.sh
  ```
* From then, to persist the environment variables required by Emscripten across sessions, you can optionally append source `~/emsdk/emsdk_env.sh` to your `~/.bashrc` file. This ensures **emscripten** is initialized automatically each time a new WSL2 terminal session is started.
  
* #⚠️ **CAUTION**: The `/mnt/...` directory represents a mounted Windows file system within the WSL2 environment, exposing Windows drives (e.g., C:) through a Linux-compatible path. While accessible via POSIX commands, this mount operates outside the WSL2 virtualized Linux file system and introduces I/O overhead. For optimal performance, especially during builds and toolchain operations, use the native WSL2 filesystem (e.g., `/home/<user>/...`) to avoid latency and permission inconsistencies.
---
### Setting up necessary build components

* This example walks through integrating **Libsodium** for AES-256-GCM encryption within a WebAssembly toolchain. It outlines the process of compiling a native C/C++ library into a wasm-compatible format and organizing the output for use in JavaScript applications or Spring Boot projects.




### Executing program

* How to run the program
* Step-by-step bullets
```
code blocks for commands
```

## Help

Any advise for common problems or issues.
```
command to run if program contains helper info
```

## Authors

Contributors names and contact info

ex. Dominique Pizzie  
ex. [@DomPizzie](https://twitter.com/dompizzie)

## Version History

* 0.2
    * Various bug fixes and optimizations
    * See [commit change]() or See [release history]()
* 0.1
    * Initial Release

## License

This project is licensed under the [NAME HERE] License - see the LICENSE.md file for details

## Acknowledgments

Inspiration, code snippets, etc.
* [awesome-readme](https://github.com/matiassingers/awesome-readme)
* [PurpleBooth](https://gist.github.com/PurpleBooth/109311bb0361f32d87a2)
* [dbader](https://github.com/dbader/readme-template)
* [zenorocha](https://gist.github.com/zenorocha/4526327)
* [fvcproductions](https://gist.github.com/fvcproductions/1bfc2d4aecb01a834b46)
