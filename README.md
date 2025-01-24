# EgoNav Docker

## Getting started
Paste in your ghp key in the dockerfile
Start docker desktop
Go to terminal
```bash
docker build -t smartbelt .
```
Wait for it to finish
And then
```bash
docker run -d --shm-size=512m -p 6080:80 --platform=linux/amd64 smartbelt
```
Go to browser http://localhost:6080 to use
