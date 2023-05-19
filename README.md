This heroku buildpack adds aspell for Ubuntu 22.

It was created by compiling [aspell](http://aspell.net/) from source in a Docker container running Ubuntu 22.04.2 LTS, using `/app` as the installation directory to mimic a running heroku env.

The file `apsell.tar.gz` was created as follows:

```bash
# Setup
mkdir /app
ASPELL_VERSION="aspell-0.60.8"
ASPELL_SOURCE="https://ftp.gnu.org/gnu/aspell/$ASPELL_VERSION.tar.gz"
BUILD_DIR=/app
CACHE_DIR="/tmp/aspell"
mkdir -p $BUILD_DIR/vendor
mkdir -p $BUILD_DIR/vendor/aspell
apt update
apt install -y curl gcc g++ build-essential
mkdir $CACHE_DIR

# Make binaries
curl -L $ASPELL_SOURCE -o "$CACHE_DIR/$ASPELL_VERSION.tar.gz"
cd tmp/aspell/
tar -xf "${CACHE_DIR}/$ASPELL_VERSION.tar.gz" -C ${CACHE_DIR} 
cd aspell-0.60.8/
./configure --enable-static --prefix="$BUILD_DIR/vendor/aspell" 
make
make install

# Make english dictionnaries
curl -L $DICT_SOURCE -o "$CACHE_DIR/$DICT_VERSION.tar.bz2" 
tar -xf "${CACHE_DIR}/$DICT_VERSION.tar.bz2" -C ${CACHE_DIR} 
cd "${CACHE_DIR}/$DICT_VERSION"
export PATH=$PATH:/app/vendor/aspell/bin
./configure --vars DESTDIR="" \
  ASPELL_FLAGS="--dict-dir='$BUILD_DIR/vendor/aspell/lib/aspell-0.60' --data-dir='$BUILD_DIR/vendor/aspell/lib/aspell-0.60'"\
  ASPELL="$BUILD_DIR/vendor/aspell/bin/aspell" \
  PREZIP="$BUILD_DIR/vendor/aspell/bin/prezip" 
make
make install

# Save everything in a tarball
cd /app/vendor/
tar zcvf aspell.tar.gz aspell/
``` 
