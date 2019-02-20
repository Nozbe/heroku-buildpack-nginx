### Brotli

We use Brotli to compress our static resources (CSS and JS). We built our own binary so that it can run on Heroku without issue.

### Versions

* Brotli: 1.0.7

### Building a new Brotli binary

Download and install [Docker](https://www.docker.com/)

From your command line:
```
# Pull the Heroku build environment image you want to use
docker pull heroku/heroku:18-build

# Run the Heroku image
docker run -it heroku/heroku:18-build
```

From the container's terminal you're now connected to:
```
# Download Brotli
curl -O -L https://github.com/google/brotli/archive/v1.0.7.zip
unzip v1.0.7.zip

# Build Brotli
cd brotli-1.0.7
make

# Zip Brotli
tar -czvf brotli.tar.gz bin

# Don't exit the container until after you've copied the zip file out of it below
```

From your command line:
```
# List running containers
docker ps

# Copy the zip file you built off the running container
docker cp cbbe1368db40:/brotli-1.0.7/brotli.tar.gz .
```
