## Arch Linux PKGBUILDs for AARCH64 & SBCs

This is a collection of Arch Linux PKGBUILDs I am experimenting with (as well as trying to build a [pacman repository](https://github.com/kwankiu/PKGBUILDs/releases/tag/experimental)).

### Branches
To keep each branch clean and easy to maintain, each branch serve different purpose.

- [main](https://github.com/kwankiu/PKGBUILDs/tree/main) - This is a repository builder containing the config and a script to build and serve a simple pacman repository.
- [rockchip](https://github.com/kwankiu/PKGBUILDs/tree/rockchip) - This branch stores PKGBUILDs for packages related to Rockchip
- [asahi](https://github.com/kwankiu/PKGBUILDs/tree/asahi) - This branch stores PKGBUILDs for packages related to Asahi-Linux (Apple Silicon)

### Using my Experimental Pacman Repository
**These are experimental packages for testing only.**

**Notes:** If a package / dependency is hosted on https://github.com/7Ji/archrepo I will not host a duplicated package on this repository. Therefore, make sure to add 7Ji's archrepo to your system as well for full experience.

As every package in this repo is signed with my PGP key, you must trust the repo before attempting to install any package.

#### Direct trust
import my signing key:
```
sudo pacman-key --recv-keys B669E3B56B3DC918
sudo pacman-key --lsign B669E3B56B3DC918
```
add the following session in your /etc/pacman.conf:
```
[experimental]
Server = https://github.com/kwankiu/PKGBUILDs/releases/download/experimental
```
#### Bypass the signature
If you encounter gpg signature issue, you may bypass / disable the signature by editing your /etc/pacman.conf like this to disable it:
```
[experimental]
SigLevel = Never
Server = https://github.com/kwankiu/PKGBUILDs/releases/download/experimental
```

### Building and Installing a package
If you just want to build / install a single package, there are many ways you can do that:
For example we want to build 	`mesa-panfrost-git`:

#### Method 1: Using [ACU](https://github.com/kwankiu/acu)
Add the branch of this repository to ACU:
```
acu rem set rockchip https://github.com/kwankiu/PKGBUILDs --branch=rockchip
```
Fetch the repository:
```
acu update
```
Build and Install the package:
```
acu install mesa-panfrost-git
```
#### Method 2: Using [AGR](https://github.com/hbiyik/agr)
Add the branch of this repository to AGR:
```
agr rem set rockchip https://github.com/kwankiu/PKGBUILDs.git --branch rockchip
```
Build and Install the package:
```
agr install mesa-panfrost-git
```

#### Method 3: Using git and makepkg

Simply clone this repository (and switch to the branch of the target pkgbuild):
```
git clone https://github.com/kwankiu/PKGBUILDs.git -b rockchip
```
Now cd to the PKGBUILD's source of the cloned repository:
```
cd PKGBUILDs/mesa-panfrost-git
```
Now (make sure you are using a user account instead of root) we can build and install the package with :
```
makepkg -si
```
OR
build (without installing) the package:
```
makepkg -s
```

### Building packages and creating your own repository

We can build multiple packages in a clean, fast and efficient way using [ARB](https://github.com/7Ji/arb).

The main branch of this repository contains a simple build script (build.sh) that allows you to build a pacman repository easily from building the packages to signing the packages as well as adding them to a pacman db. Then you can simply upload the generated `repo` directory to a hosting provider or GitHub release.

#### To begin, simply clone this repository:
```
git clone https://github.com/kwankiu/PKGBUILDs.git
```
Now cd to the cloned repository:
```
cd PKGBUILDs
```
#### Add your signing key
Add your private key to gpg (or [create one](https://gist.github.com/elieux/fad9451bbfc4ddb5cde7) if you dont already have a key): 
Notes: Enter your passphrase whenever prompted to do so.
```
gpg --import myprivkey.asc
```

Import your signing key to pacman if you haven't:
```
sudo pacman-key --recv-keys YOURKEY
sudo pacman-key --lsign YOURKEY
```
#### (Optional) Change the repository name:
Edit `build.sh`:
Replace
```
repo_name="experimental"
```
with your preferred repo name
```
repo_name="YOUR_REPO_NAME"
```
#### (Tips) Building on x86_64 or if your conf outputs .pkg.tar.zst instead of .pkg.tar.xz
Edit `build.sh`:
Uncomment the zst one and comment the xz one:
```
repo-add -s -v -n db/${repo_name}.db.tar.xz *.pkg.tar.zst
#repo-add -s -v -n db/${repo_name}.db.tar.xz *.pkg.tar.xz
```
#### Start building your repository
You may also want to customize or use your own config yaml, ARB config yaml are documented [here](https://github.com/7Ji/arch_repo_builder#config).

Make sure you have permission to run the build script:
```
sudo chmod +x build.sh
```
To build all packages (from all yaml) at once:
```
./build.sh
```
OR
To build packages from a specified yaml (for example `rockchip.yaml`):
```
./build.sh rockchip
```

Once it is done, the repository are generated at the `repo` directory, you can simply upload everything from the generated directory to a hosting provider or GitHub release.
