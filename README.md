# Windows Kubernetes Samples

This is a group of sample apps that I use to build demos. Check the history to get an idea of when each was used last.


## RazorPagesMovie

> This is based on the samples from https://github.com/aspnet/Docs, specifically https://github.com/aspnet/Docs/tree/master/aspnetcore/tutorials/razor-pages/razor-pages-start/sample/RazorPagesMovie . I added the README.md and scripts needed to build locally with Docker for Windows, then deploy on Kubernetes.

### Build & test locally

Build it - `docker build --pull -t razorpagesmovie .`

Test it - `docker run --name razorpagesmovie --rm -it -p 8000:80 razorpagesmovie`, then browse to (http://127.0.0.1:8000/Movies)


## Random Tips

- Greg Bayer has a good blog post on [Moving FIles from one Git Repository to Another, Preserving History](https://gbayer.com/development/moving-files-from-one-git-repository-to-another-preserving-history/) which I used to seed this repo with just the sample code, instead of the full .Net doc set :)