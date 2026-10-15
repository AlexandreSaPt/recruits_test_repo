# Software Task Repository


## Supported systems

The C++ code uses POSIX sockets (`<netinet/in.h>`, `<poll.h>`, `<sys/socket.h>`), so it builds on **Linux and macOS**. On **Windows use WSL2** (Ubuntu) and follow the Linux instructions inside it.

---

## 1. Python side (`vm_server.py`)

### 1.1 Requirements

- Python 3 (tested with **3.12**; any recent 3.x release should work)
- The libraries in `requirements.txt`: `websockets` and `foxglove-websocket`

Check your Python:

```bash
python3 --version
```

### 1.2 Install Python and venv support

**Ubuntu / Debian / WSL**

```bash
sudo apt update
sudo apt install -y python3 python3-venv python3-pip
```

**macOS** (with [Homebrew](https://brew.sh))

```bash
brew install python
```

**Fedora**

```bash
sudo dnf install -y python3 python3-pip
```

### 1.3 Create the `recruits_env` environment (manual way)

From the project folder:

```bash
# 1. create the environment
python3 -m venv recruits_env

# 2. activate it (do this in every new terminal)
source recruits_env/bin/activate

# 3. install the libraries
python -m pip install --upgrade pip
python -m pip install -r requirements.txt
```

Your prompt now starts with `(recruits_env)`. To leave the environment later, run `deactivate`.

### 1.4 Run the VM

```bash
source recruits_env/bin/activate
python vm_server.py
```

You should see:

```
Starting Vehicle Model backend...
🌍 Web UI is running! Open http://localhost:8000 in your browser
🦊 Foxglove streaming live at ws://localhost:8765
```

Open <http://localhost:8000> to interact with the VM. Stop it with `Ctrl+C`.


### 1.5 The easy way: Makefile

`Makefile.python` does steps 1.3 and 1.4 for you (it creates `recruits_env` the first time and reinstalls if `requirements.txt` changes):

```bash
make -f Makefile.python          # create env if needed + run vm_server.py
make -f Makefile.python env      # only create the env and install libraries
make -f Makefile.python run      # run vm_server.py
make -f Makefile.python clean    # delete recruits_env
```

---

## 2. C++ side (`main.cpp`)

### 2.1 Requirements

| Tool / library | Why |
|---|---|
| `g++` with C++17 support (GCC 8+ / Clang 7+) | compiler |
| `make` | to use `Makefile.cpp` (optional) |
| **nlohmann-json** (header-only) | JSON parsing/building, included as `<nlohmann/json.hpp>` |

No other libraries are needed: sockets come from the operating system.

### 2.2 Install the compiler and libraries

**Ubuntu / Debian / WSL**

```bash
sudo apt update
sudo apt install -y build-essential nlohmann-json3-dev
```

`build-essential` installs `g++` and `make`. `nlohmann-json3-dev` installs the headers into `/usr/include/nlohmann`.

**Fedora / RHEL**

```bash
sudo dnf install -y gcc-c++ make json-devel
```

**macOS**

```bash
xcode-select --install          # compiler (clang++, available as g++) and make
brew install nlohmann-json
```

Verify:

```bash
g++ --version
ls /usr/include/nlohmann/json.hpp          # Linux
ls "$(brew --prefix)/include/nlohmann"     # macOS
```

### 2.3 Build and run (manual way)

```bash
g++ -std=c++17 -O2 main.cpp -o mcu
./mcu
```

On macOS with Homebrew add the include path: `g++ -std=c++17 -O2 -I"$(brew --prefix)/include" main.cpp -o mcu`.

Expected output when the VM is running:

```
MCU running. Listening on 9000, sending to 9001...
Setup complete.
packets=201  torque=0 Nm  TS=0  R2D=0  relay=1  brake_light=0
```

If the VM is not running you will see `Waiting for the VM on UDP 9000...`. Stop the MCU with `Ctrl+C`.

### 2.4 The easy way: Makefile

```bash
make -f Makefile.cpp             # compile main.cpp -> ./mcu
make -f Makefile.cpp run         # compile (only if a source changed) and run ./mcu
make -f Makefile.cpp clean       # delete ./mcu
```

`Makefile.cpp` rebuilds automatically when `main.cpp`, `recruits.hpp` or `vm_io.hpp` change, and adds the Homebrew include path on macOS. You can override the compiler or flags, e.g. `make -f Makefile.cpp CXX=clang++`.

---

## 3. Running everything together

Use **two terminals** in the project folder. Start the VM **first**.

**Terminal 1 – VM**

```bash
make -f Makefile.python
```

**Terminal 2 – MCU**

```bash
make -f Makefile.cpp run
```

Then open <http://localhost:8000>.