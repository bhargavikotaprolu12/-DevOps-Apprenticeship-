
Run: docker history nginx
docker history nginx shows the layer history of the nginx image, meaning the commands and layers used to build it.
Then
docker image inspect nginx
Observe
    • Layers
    • Size
    • Metadata
The docker history command displays the layer history of a Docker image. Each row represents a layer created during the image build process, usually corresponding to a Dockerfile instruction. Instructions like RUN and COPY often increase image size because they modify the filesystem, while instructions like CMD, ENTRYPOINT, ENV, LABEL, and EXPOSE mainly store metadata and usually don't add significant size. Docker uses these layers for efficient caching, faster builds, and image sharing.

now run your own java image and inspect it.
