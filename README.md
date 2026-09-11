# Környezet OpenCL gyakorlatokhoz

Ez az oldal röviden ismerteti az OpenCL gyakorlatokhoz szükséges környezet létrehozását saját gépen.

A hoszt operációs rendszer lehet natív Linux (preferált az Ubuntu 24.04 vagy ennek valamilyen származéka) vagy Windows alatt a Windows Subsystem for Linux (WSL2). Tartalomjegyzék:

- Környezet
  - [Natív Linux](#natv-linux)
  - [WSL2](#wsl2)
  - [Termux](#termux)
  - [OpenCL](#opencl)
    - [AMD](#amd)
    - [Intel](#intel)
    - [NVIDIA](#nvidia)
    - [MESA](#mesa)
    - [PoCL](#pocl)
    - [Termux](#termux-1)
  - [Docker](#docker)
  - [Visual Studio Code](#visual-studio-code)
- [VSCode, fordítás, futtatás](#vscode-fordts-futtats)

# Környezet

## Natív Linux

Natív Linux használata esetén javasolt az Ubuntu 24.04 vagy annak valamelyik leszármazottja.

## WSL2

A WSL2 a Microsoft Hyper-V virtualizációs technológiáján futó teljes értékű Linux. WSL környezet installálás:

```bash
wsl --install --no-distribution​
```

Elérhető disztribúciók listázásda, Ubuntu 24.04 installálás:

```bash
wsl --list --online
wsl --install Ubuntu-24.04
```

(WSL installálás után a Windows is virtualizálva fut, így egyes - tipikusan CPU tuning - alkalmazások, amik közvetlen regiszter hozzáférést igénylenek, nem fognak működni.)

## Termux

A [Termux](https://termux.dev/en/) egy Android-on megvalósított Linux terminal. Közvetlenül nem tud Linux futtatható állományokat futtatni, de újrafordítással jól használható. Friss verzió installálása:

- [Instaláljuk az F-Droid appot](https://f-droid.org/en/​)
- Az F-Droid-ból instaláljuk a "Termux terminal emulator with packages​"-t
- Ha nem tudunk F-Droid-t installálni, próbálkozhatunk a Play Store-ban levő Termux-szal is, vagy közvetlenül az APK letöltésével.

Fríssités, SSH szerver installálás (és automatikus indítás) és jelszó beállítás:

```bash
apt update​
apt upgrade​
apt –y install openssh​
passwd​
```

Indítsuk el az SSH daemon-t:

```bash
sshd
```

Ezután tudunk csatlakozni SSH-val a 8022 porton root user-ként. Az IP címet ifconfig-gal meg tudjuk nézni.

Enegedélyezzük, hogy az sshd automatikusan elinduljon.

```bash
apt -y install termux-services
sv-enable sshd
```

## OpenCL

Natív Linux-ban AMD és Intel GPU használata esetén installálhatjuk a gyártó OpenCL driverét, vagy használhatjuk a MESA Rusticl drivert. NVIDIA esetén a gyártó drivere az egyetlen opció.

WSL2-ben az AMD és az Intel kínál OpenCL gyorsítást (gyártói driver kell), az NVIDIA csak CUDA-t. Így ott a PoCL a megoldás, CUDA backend-del.

Termux esetén OpenCL haználatára a telefonon remélhetőleg meglévő Android OpenCL drivere kínál lehetőséget.

Amennyiben nincs OpenCL képes GPU-nk vagy VMware-t/VirtualBox-t használunk, a PoCL CPU backend-jét használva tudunk OpenCL-t használni.

Amennyiben nem tudjuk a GPU-nk architektúráját, a [Techpowerup adatbázisa segít](https://www.techpowerup.com/gpu-specs/).

Hasznos alkalmazás a clinfo:

```bash
sudo apt update​
sudo apt -y install clinfo​
```

OpenCL eszközök listázása:

```bash
clinfo -l
```

Az összes eszköz részletes tulajdonságainak listázása:

```bash
clinfo
```

További hasznos alkalmazás a clpeak, amely az OpenCL eszköz maximális teljesítményét méri.

```bash
sudo apt install clpeak
```

### AMD

Az AMD gyári driverének (amdgpu) eszköz támogatása elég katyvasz és/vagy limitált. Azaktuális driver a ROCr, ez GCN 5.0-tól (Vega 11) támogat eszközöket, a ROCm része.

- A [legfrissebb ROCm verzió a 10.0.0](https://rocm.docs.amd.com/​), de ez hivatalosan csak a legújabb GPU-kat támogatja.
  - Jóval szélesebb a [támogatott GPU-k köre](https://github.com/ROCm/TheRock/blob/main/SUPPORTED_GPUS.md) a [TheRock](https://github.com/ROCm/TheRock/blob/main/RELEASES.md​)-nak, ami kb a ROCm nightly build-je.
  - A 10.0.0 WSL2-t is támogat.
- Amennyiben a 10.0.0 nem opció, a javasolt régebbi verziók a 7.2.4 és a 6.4.4.
  - [7.2.4 installálási útmutató](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-7.2.4/install/quick-start.html)
  - [6.4.3 installálsi útmutató](https://rocm.docs.amd.com/projects/install-on-linux/en/docs-6.4.3/install/quick-start.html)
    - [6.4.4 fájlok](https://repo.radeon.com/amdgpu-install/6.4.4/ubuntu/noble/)

A GCN 5.0-nál régebbi eszközöket a legacy OpenCL driver támogatja, ez az [amdgpu 5.7.1](https://repo.radeon.com/amdgpu-install/5.7.1/ubuntu/) verziójában volt elérhető, csak Ubuntu 22.04-ig. Installálás:

```bash
sudo amdgpu-install --usecase=dkms,opencl --opencl=legacy --accept-eula​
```

Alternatíva a MESA radeonsi driver használata rusticl-lel.

### Intel

Az Intel a Gen8 vagy újabb GPU-khoz kínál driver-t ([Intel NEO](https://github.com/intel/compute-runtime)), ez jobbára minden disztribúcióban benne van:

```bash
sudo apt install intel-opencl-icd​
```

Alternatíva lehet a MESA iris driver rusticl-lel, ez ugyancsak Gen8 és újabb GPU-kat támogat.

### NVIDIA

Natív Linux használata esetén minden disztribúció tartalmaz NVIDIA drivert, aminek része az OpenCL.

- Turing és újabb GPU-k esetén a legfrissebb (590.x, 595.x, 600.x, 610.x) elérhető driverek megfelelők, ez disztribúció függő.
- A Kepler GPU-kat a 470.x driverek támogatják
- Pascal, Maxwell architektúrák: 580.x
- Fermi GPU-khoz a 390.x driver használható, de csak régebbi kernellel

WSL2 alatt csak CUDA driver van, itt a PoCL CUDA backend-je használható. Ugyanez igaz az NVIDIA SoC-okra is.

### MESA

A GPU MESA driverének használatához az szükséges, hogy a gyártói driver-t ne installáljuk, általában ez az alapbeállítás.

MESA OpenCL installálás:

```bash
sudo apt install -y mesa-opencl-icd​
```

A Rusticl használatát külön engedélyezni kell a **RUSTICL_ENABLE** környezeti változóval adott driver(ek)hez, pl radeonsi-hez:

```bash
export RUSTICL_ENABLE=radeonsi
```

Mivel a MESA és a Rusticl elég gyorsan fejlődik, érdemes lehet a disztribúcióban található régebbi verziót az aktuálisra cserélni, legfeljebb több lesz a bug :)

```bash
sudo add-apt-repository ppa:oibaf/graphics-drivers​
sudo apt update​
sudo apt upgrade​
```

### PoCL

A PoCL tipikusan régebbi, CPU backend-t tartalmazó verzója a disztribúciók jelentős részében rendelkezésre áll:

```bash
sudo apt install -y pocl-opencl-icd​
```

Amennyiben szeretnénk CUDA backend-t, a legjobb, ha magunk fordítunk. Ehhez kelleni fog a CUDA Tollkit, [installálási útmutató WSL2-höz itt érhető el](https://developer.nvidia.com/cuda-13-3-1-download-archive?target_os=Linux&target_arch=x86_64&Distribution=WSL-Ubuntu&target_version=2.0). A CUDA toolkit installálsa után ellenőrizzük, hogy a GPU látszik-e WSL2-ben:

```bash
nvidia-smi​
```

PoCL 7.2 fordítás és installálás:

```bash
export LLVM_VERSION=20​
sudo apt install -y build-essential ocl-icd-libopencl1 cmake git \
pkg-config libclang-${LLVM_VERSION}-dev clang-${LLVM_VERSION} \
llvm-${LLVM_VERSION} make ninja-build ocl-icd-libopencl1 \
ocl-icd-dev ocl-icd-opencl-dev libhwloc-dev zlib1g zlib1g-dev \
clinfo dialog apt-utils libxml2-dev libclang-cpp${LLVM_VERSION}-dev \
libclang-cpp${LLVM_VERSION} llvm-${LLVM_VERSION}-dev libncurses6​
cd ~/Downloads​
git clone -b release_7_2 https://github.com/pocl/pocl.git pocl-7.2​
cd pocl-7.2​
mkdir build​
cd build​
cmake -DCMAKE_INSTALL_PREFIX=/opt/pocl-7.2 -DCMAKE_BUILD_TYPE=Release \
-DCMAKE_CXX_FLAGS="-funroll-loops -march=native -L/usr/lib/wsl/lib" \
-DCMAKE_C_FLAGS="-funroll-loops -march=native -L/usr/lib/wsl/lib" -DENABLE_CUDA=ON ..
make​
sudo make install​
sudo mkdir -p /etc/OpenCL/vendors/​
sudo touch /etc/OpenCL/vendors/pocl-7.2.icd​
echo "/opt/pocl-7.2/lib/libpocl.so" | sudo tee --append /etc/OpenCL/vendors/pocl-7.2.icd​
```

Ezután a clinfo remélhetőleg egy CPU és egy GPU eszközt listáz.

### Termux

Az Android OpenCL driverét szeretnénk használni:

```bash
apt update​
apt upgrade​
apt -y install clinfo clpeak opencl-vendor-driver​

```

A továbbiak GPU függőek.

ARM Mali vagy IMG PowerVR esetében a clinfo listázza a GPU-t.

Qualcomm Snapdragon esetében extra library-re van szükség:

```bash
LD_LIBRARY_PATH=/vendor/lib64 clinfo -l​

```

Ugyanez a helyzet Samsung Xclipse GPU-k esetén:

```bash
LD_LIBRARY_PATH=/vendor/lib64:/vendor/lib64/hw clinfo -l​
```

## Docker

Installáljuk a [Docker engine-t](https://docs.docker.com/engine/install/) és hajtsuk végre az [installálás utáni lépéseket](https://docs.docker.com/engine/install/linux-postinstall/) is.

## Visual Studio Code

Natív Linux esetén töltsük le a disztribúciónak megfelelő fájlt, WSL2 esetén pedig Winows alá installáljuk a VSCode-t [INNEN](https://code.visualstudio.com/download?_exp_download=fb315fc982).

Installáljuk az alábbi kiegészítéseket:

```bash
code --install-extension ms-vscode.cpptools​
code --install-extension ms-vscode.cpptools-extension-pack​
code --install-extension ms-vscode.cpptools-themes​
code --install-extension ms-vscode-remote.remote-containers​
code --install-extension ms-vscode.makefile-tools​
code --install-extension galarius.vscode-opencl​
```

# VSCode, fordítás, futtatás

### VSCode

Szerkesztőként/GUI-ként a VSCode-t használjuk, míg a fordítást Docker image-ekben hajtjuk végre. Ennek oka, hogy a Docker egyszerűen teríthető, hoszt független, és az esetleges package ütközéseket is könnyebb kezelni. A Teams csoportból töltsük le a **vscode_amd64_arm64.zip** fájlt és csomagoljuk ki valahova, pl. ~/heterogen. A létrejövő könyvtár struktúra:

```plaintext
-- heterogen
   |-- vscode_amd64
       |-- .devcontainer
       |-- helloworld
   |-- vscode_arm64
       |-- .devcontainer
       |-- helloworld
```

A .devcontainer konyvtárak tartalmazzák a dockerfile-t, ami alapján az adott architektúrára történő fordításhoz szükséges Docker image generálódik. Ezzel a könyvtárral egy szinten helyezkednek el a C++ projektjeink könyvtárai, ebből a ZIP-ben egy van, a helloworld.

Adott C++ projekt megnyitása a következő. Indítsuk el a VSCode-t a megfelelő architetúra könyvtárából (pl. ~/heterogen/vscode_amd64):

```bash
code .
```

A VScode észlelni fogja a .devcontainer könyvtárat, és felajánlja, hogy megnyitja a konténerben az éppen megnyitott könyvtárat - erre nyomjunk igen-t. Ezután nyithatjuk meg a projetet tartalmazó könytárat a File/Open Folder menüben.

(Megj. Mint látható, a projekt könyvtárak duplikálva vannak. Ennél szebb lenne, ha ezek egy külön könyvtárban lennének, és az architektúrák alá ezeket symlink-elnénk. Csak azért nem ezt tesszük, hogy a WSL2-t haználók tudják a hoszt Windows fájlkezelő alkalmazásait használni.)

### Fordítás

A fordítást makefile-lal hajtjuk végre, így

- A make futtaható terminal-ból.
- A VSCode-ba installáltuk a Makefile Tools-t, így a bal oldali menüben azt kiválasztva, majd Build target opciót All-ra állítva a Build ikonnal is tudunk fordítani.

### Futatás

Ahhoz, hogy a lefordított alkalmazásainkat futtatni tudjuk, a hoszt gépen az alábbi package-ekre lesz szükség.

```bash
sudo apt update​
sudo apt -y install libgomp1​
sudo apt -y install libsndfile1​
sudo apt -y install libdevil1c2​
```
