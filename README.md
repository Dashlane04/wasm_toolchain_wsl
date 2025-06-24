# wasm_toolchain_wsl
🧰 **A workaround toolchain for converting C/C++ code and libraries into wasm for web development with compiled js scripts.**

## Description

A toolchain targeted for **Windows** users navigating through system using **WSL2** (Windows Subsystem for Linux). This toolchain helps convert native c++ code and libraries into **wasm** (Web Assembly) using **Emscripten** and **autoconf automake libtool**. Furthermore, it can be used with frameworks like SpringBoot for client-side javascript delivery. 

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
  * Since the file systems for Windows and Linux are separated in a WSL2 environment, we streamline the workflow by running VSCode directly within the WSL2 Linux   context. This ensures consistent path resolution (particularly in **c_cpp_properties.json**) and avoids cross-platform confusion.
    To do this, navigate to your project directory inside the WSL terminal and launch VSCode in full Linux mode:
    
    ```
    cd /home/user_name/project_dir
    ```
    Then, execute:
    ```
    code .
    ```
    This should open the targeted VSCode project in full linux mode. You can verify by looking at the bottom left of the VSCode window, which states the name of the current linux distro.
    
    
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
### 📚 Setting up necessary build components

* This example walks through integrating **Libsodium** for AES-256-GCM encryption within a WebAssembly toolchain. It outlines the process of compiling a native C++ library into a wasm-compatible format and organizing the output for use in JavaScript applications or Spring Boot projects. This is good for secure message transmission between clients for a realtime chatapp because encryption/decryption is handled with near native C/C++ performance.

* After installing **Libsodium** in the `/home/<user>/...` directory, check if the directory already contains a `configure` file or not. This file is crucial for creating the `Makefile` as well as automatically configuring software source code packages to adapt to Unix-like systems. If not found, run the following bash code in the **Libsodium** package directory to generate:

  ```
  ./autogen.sh
  ```

  The `configure` file should now exist in the **Libsodium** directory.

* Get the official **Libsodium** package via:

  ```
  wget https://download.libsodium.org/libsodium/releases/libsodium-1.0.19.tar.gz
  ```
  Then unpack into the same user root directory:

  ```
  tar -xvzf libsodium-1.0.19.tar.gz
  ```

  After you are done with installing the package, create a new directory to store the later compiled wasm-compatible format of the package.

  ```
  mkdir libsodium-wasm
  ```

* To compile a C++ file containing ``#include <sodium.h>`` and ``#include <emscripten.h>`` into WebAssembly using **Emscripten**, the **emcc compiler** needs to be explicitly directed to the precompiled, WASM-compatible version of **Libsodium**. ##⚠️ This approach is necessary for any package used in conjunction with the emscripten header.

* Now run these to compile the package into wasm, which generates the **include** and **lib** directory:

  ```
  emconfigure ./configure --prefix=/home/<user>/libsodium-wasm
  emmake make -j$(nproc)
  emmake make install
  ```

* A proper **c_cpp_properties.json** is needed if you want the Intellisense in VSCode to recognise the components. This file is not actually used by the ``emcc`` build method involving **emscripten** that this toolchain requires, but it is needed by the native VSCode C++ compiling process. However, it's good for clarification and readability. The file would look something like this:

  ```
  {
  "configurations": [
    {
      "name": "WSL",
      "includePath": [
        "${workspaceFolder}/**",
        "/home/<user>/libsodium-wasm/include",
        "/home/<user>/emsdk/upstream/emscripten/system/include",
        "/home/<user>/emsdk/upstream/emscripten/cache/sysroot/include"
      ],
      "defines": [
        "_DEBUG",
        "UNICODE",
        "_UNICODE"
      ],
      "compilerPath": "/usr/bin/clang-19",    (you can also use gcc or g++)
      "cStandard": "c17",
      "cppStandard": "c++17",
      "intelliSenseMode": "linux-gcc-x64"
    }
  ],
  "version": 4
  }

  ```
You can access this file in VSCode by opening the **Command Palette** in **Settings**, then searching for and selecting `Preferences: Open User Settings (JSON)`. This configuration file defines the appropriate include paths for the C++ toolchain, aligned with the WSL2 virtual Linux file system, ensuring IntelliSense and build tools resolve headers correctly.

---
### 🛠 Compiling the C++ file in VSCode

* Navigate to the VSCode project directory and execute the following, which outputs a corresponding .wasm and .js file from the selected .cpp file. **You can also save this script in a bash file to avoid manually repeating the same process**:
```
emcc test.cpp \
  -I/home/<user>/libsodium-wasm/include \        
  -L/home/<user>/libsodium-wasm/lib -lsodium \
  -s EXPORTED_FUNCTIONS='["func_name"]' \
  -s EXPORTED_RUNTIME_METHODS='["ccall"]' \
  -s NO_EXIT_RUNTIME=1 \
  -o test.js
```
* Options:
* ``-I/home/<user>/libsodium-wasm/include`` is the relative path for the **Libsodium** precompiled wasm-compatible **include**.
  
* ``-L/home/<user>/libsodium-wasm/lib `` is the relative path for the **Libsodium** precompiled wasm-compatible **library**.
  
* ``-lsodium``: the ``-l<name>`` flag is part of the linking process. This option tells the linker to resolve references to functions declared in headers by associating them with precompiled implementations found in libraries like **lib<name>.a** or **lib<name>.so**. For example, when you use the ``sodium_init()`` function to initialize the **Libsodium** library from the ``<sodium.h>`` header, the linker will try to map the corresponding precompiled wasm-compatible code to the function. ⚠️ **This is a crucial step and must be done correctly!**
  
* ``-s EXPORTED_FUNCTIONS='["func_name"]'`` tells Emscripten to explicitly export the C++ function named func_name (and any other functions listed) from the compiled WebAssembly module, making it callable directly from JavaScript.
  
* ``-s EXPORTED_RUNTIME_METHODS='["ccall"]'`` instructs Emscripten to include specific methods from its JavaScript "glue code" runtime into the final output. ``ccall`` is a common **Emscripten** runtime method that provides a convenient way for JavaScript to call C/C++ functions, handling argument type conversions and return values. Check this url for more info: [emcripten-(connecting_cpp_and_js)](https://emscripten.org/docs/porting/connecting_cpp_and_javascript/Interacting-with-code.html)

* ``-s NO_EXIT_RUNTIME=1``: This flag prevents the **Emscripten** runtime from automatically shutting down after the main function completes or ``exit()`` is called, ensuring that exported C/C++ functions remain callable from JavaScript for the entire duration of the web page's lifecycle.

---
### C++ file
* Your C++ file would look something like this:

  ```
   #include <iostream>
   #include <emscripten.h>
   #include <sodium.h>
   using namespace std;

   EMSCRIPTEN_KEEPALIVE
   extern "C"{
   int main() { 
    if (sodium_init() < 0) {
        cout << "Libsodium init failed!" << endl;
        return 1;
    }
    cout << "Libsodium is ready!" << endl;
    return 0;
   }
   }
  ```





 

