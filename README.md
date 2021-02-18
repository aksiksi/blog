# blog

## Build & Serve

Easiest way is to use the Jekyll Docker image:

```
sudo docker run --rm -it -p 4000:4000 --volume="$PWD:/srv/jekyll" jekyll/jekyll jekyll serve
```

This builds the blog and runs a server within the Docker container. The blog will be rebuilt on any changes.
