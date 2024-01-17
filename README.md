# docker-vnc-base

This repository holds a Docker image configured to run a VNC server compatible with 3D acceleration using Nvidia GPUs.

## Instructions

Build the image locally:

```bash
cd src
docker build -t vnc-base . 
```

Run the image locally:

```bash
docker run --rm -it --gpus all --shm-size=512m -p 6901:6901 -e VNC_DISABLE_AUTH=true --user root vnc-base
```

Open a browser and go to: https://localhost:6901

## Configuration

Available environment variables:

| Variable                                  | Description                                                                                   | Default     |
|-------------------------------------------|-----------------------------------------------------------------------------------------------|-------------|
| VNC_DISABLE_AUTH                          | Disable VNC authentication (no user/password required, use with caution)                      | false       |
| VNC_PW                                    | Password for the basic auth (the user is always: "user")                                      | vncpassword |
| VNC_PORT                                  | VNC port                                                                                      | 6901        |
| VNC_ALLOW_CLIENT_TO_OVERRIDE_VNC_SETTINGS | Enable/disable the client to override the VNC settings, it does not have effect on the web UI | false       |


## Third-party copyright notices

License for files `.devcontainer/library-scripts/common-debian.sh` and `.devcontainer/library-scripts/docker-debian.sh`

    MIT License

    Copyright (c) Microsoft Corporation. All rights reserved.

    Permission is hereby granted, free of charge, to any person obtaining a copy
    of this software and associated documentation files (the "Software"), to deal
    in the Software without restriction, including without limitation the rights
    to use, copy, modify, merge, publish, distribute, sublicense, and/or sell
    copies of the Software, and to permit persons to whom the Software is
    furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all
    copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR
    IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY,
    FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE
    AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER
    LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM,
    OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE
    SOFTWARE

Files under the `src` directory are based on [Kasm Core Ubuntu Image](https://github.com/kasmtech/workspaces-core-images), which is licensed under the following terms:

    # Disclaimer

    This license applies only to the source code that is directly maintained in this git repository, it does not extend to dependencies from outside of this repository, to include other projects owned and/or maintained by Kasm Technologies.

    ## License

    Copyright 2022 Kasm Technologies Inc

    Permission is hereby granted, free of charge, to any person obtaining a copy of this software and associated documentation files (the "Software"), to deal in the Software without restriction, including without limitation the rights to use, copy, modify, merge, publish, distribute, sublicense, and/or sell copies of the Software, and to permit persons to whom the Software is furnished to do so, subject to the following conditions:

    The above copyright notice and this permission notice shall be included in all copies or substantial portions of the Software.

    THE SOFTWARE IS PROVIDED "AS IS", WITHOUT WARRANTY OF ANY KIND, EXPRESS OR IMPLIED, INCLUDING BUT NOT LIMITED TO THE WARRANTIES OF MERCHANTABILITY, FITNESS FOR A PARTICULAR PURPOSE AND NONINFRINGEMENT. IN NO EVENT SHALL THE AUTHORS OR COPYRIGHT HOLDERS BE LIABLE FOR ANY CLAIM, DAMAGES OR OTHER LIABILITY, WHETHER IN AN ACTION OF CONTRACT, TORT OR OTHERWISE, ARISING FROM, OUT OF OR IN CONNECTION WITH THE SOFTWARE OR THE USE OR OTHER DEALINGS IN THE SOFTWARE.


