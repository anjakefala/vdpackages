# Misc Info 

- Homebrew relies on a git repository storing the 'formulas' to install package
- Note that in order for a package to be included in `homebrew-core`, it needs to pass the [acceptable formulae](https://docs.brew.sh/Acceptable-Formulae) requirements and it needs to pass all `brew audit --new-formula <formula>` tests
- It essentially installs packages into their own directory (Keg) within a Cellar and then symlinks those files to the relevant places in `/usr/local`
- 'Formulas' are Ruby files that define classes extending Homebrew's own `Formula` class
    - these classes serve as instructions of how to install that specific package on a user's machine
    - formula can be created with `brew create <url>` where <url> is a zip or tarball
    - formula can be installed with `brew install <formula>` and debugged with `brew install --debug --verbose <formula>
    - `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/foo.rb`
    - [Formula api](http://www.rubydoc.info/github/Homebrew/brew/master/Formula)
    - To check out a formula use `brew edit <formula>
- `Keg` -> the installation prefix of a `Formula` 
    - `/usr/local/Cellar/foo/0.1`
    - Basically, where it is literally installed
- `opt prefix` -> a symlink to the active version of a `Keg`
    - `/usr/local/opt/foo`
- `Cellar` -> where all `Kegs` are installed 
    - `/usr/local/Cellar`
    - You can `brew ls` a few of the kegs in your Cellar to see how it is all arranged
- `Tap` -> A Git repository of Formulae and/or commands
    - `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core`
- `Bottle` -> Pre-built `Keg` instead of building from source

- For a software to be 'key-only' means that it is installed in `/usr/local/Cellar` but not linked into places like `/usr/local/bin`, `/usr/local/lib`, etc. That means other software that depends on it has to be compiled with specific instructions to use the files in `/usr/local/Cellar`. That is done automatically by `brew install` when a formula specifies key-only dependencies. Formulas that specify keg-only dependencies make sure that the equivalent system libraries are not used. 

create URL [--autotools|--cmake|--meson] [--no-fetch] [--set-name name] [--set-version version] [--tap user``/``repo]: Generate a formula for the downloadable file at URL and open it in the editor. Homebrew will attempt to automatically derive the formula name and version, but if it fails, you'll have to make your own template. The wget formula serves as a simple example. For the complete API have a look at http://www.rubydoc.info/github/Homebrew/brew/master/Formula.

    If --autotools is passed, create a basic template for an Autotools-style build. If --cmake is passed, create a basic template for a CMake-style build. If --meson is passed, create a basic template for a Meson-style build.

    If --no-fetch is passed, Homebrew will not download URL to the cache and will thus not add the SHA256 to the formula for you. It will also not check the GitHub API for GitHub projects (to fill out the description and homepage).

    The options --set-name and --set-version each take an argument and allow you to explicitly set the name and version of the package you are creating.

    The option --tap takes a tap as its argument and generates the formula in the specified tap.

# Steps
- Run `brew create` with a URL to the source tarball
    - Check the Releases tab on github for the download URL

- This creates `/usr/local/Homebrew/Library/Taps/homebrew/homebrew-core/Formula/foo.rb` and opens it in your $EDITOR.

- Fill in the `homepage` section in the Formula. An https homepage is preferred, if one is available

- Summarise what the software is in the `desc` section; Note that `desc` is automatically prepended with the formula name

- Specifying other formulae as dependencies
    - 
    ```
    class Foo < Formula
        depends_on "pkg_config" => :run
        depends_on "jpeg" 
        depends_on "boost" => "with-icu"
        depends_on "readline" => :recommended
        depends_on "gtk+" => :optional
        depends_on ":xll" => :optional
     end
     ```
     - The string which follows `depends_on` (e.g. "jpeg") specifies a formula dependency
     - A symbol (e.g ':xll') specifies a Requirement which can be fulfilled by one or more formulae, casks or other system-wide installed software
     - A Hash (e.g. =>) specifies a formula dependency with soem additional information
     - Homebrew requires you to use `resource`s for all language-specific dependencies
     - https://github.com/tdsmith/homebrew-pypi-poet can help us generate resource stanzas
    - To explicitly add your Python dependencies, you need to browse PyPI to find the download URL for your resource. (e.g. here https://github.com/Homebrew/homebrew-core/blob/master/Formula/jrnl.rb)
    e.g.
    ```
    resource "pyshp" do
        url "https://pypi.python.org/blah/pyshp-1.5.0.tar.gz"
        sha1 "blah"
    end
    ```
    - Note that PyPI only gives you the MDF hash, to get the sha1 hash use `shasum` on the tarball
- Check the build
    - `brew install --verbose --debug <software>`
        - Take note of the SHA that Homebrew reports, you should add this to your formula 
    - Check the package's `README`. Check if the package installs with `./configure`, `cmake` or something else. Delete the commented out lines for the irrelevant ones
    - The first time you run, nothing should have been installed because you have to add the installation method

- Add installation directions
    ```
    def install
        resource("pyshp").stage{ system "python", *Language::Python.setup_install_args(libexec/"vendor")}
    ```
- Check for dependencies in `/usr/lib`
    - If you are dependent on software which is `keg_only`, you need to have environment variables set or special configuration flags passed to locate that software
    - You can create a `Requirement`

- Add a test to the `test do` block of the formula. This will be run by `brew test projectName`

- Homebrew expects to find manual pages in #{prefix}/share/man/...
    - Add a `--mandir=#{man}` to the ./configure line if needed

- Audit the formula
    - `brew audit --new-formula software` -> makes it more likely to be quickly accepted

- Fork homebrew-core

- Submit your package to homebrew-core
    ```
    brew update
    cd $(brew --repo homebrew/core)
    git checkout -b <descriptive-name>
    git add Formula/visidata.rb
    git commit
    ```
    - commit message "visidata v0.98.1 (new formula)" (two spaces and then a more detailed expalnation)
- Open a pull request for your changes
