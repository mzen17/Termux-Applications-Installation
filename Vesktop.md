# Vesktop [Discord]
Status: Partially working.

## Quick bits: Why Vesktop?
Vesktop is an open source discord client, unlike the official discord app. This makes it great for recompilation to make sure things work. In addition, it does come with the benefits of vencord, and is what I use on my standard Arch Linux boxes. It is also a great stress-test for Termux.

# Build steps
This one was rather tricky. There were two things that I tried:
1. Downloading the Linux tarball, extract, and trying to use glibc to run it. This did not work because of some weird bug with missing libffmpeg.so libraries, and it did not work even after compiling ffmpeg from source against glibc.

2. Building it from source [successful]. I was able to get this working through a painful process.

## Buildling from source
I first cloned the Vesktop repository, and then ran `npm install -g pnpm`. After this, `pnpm i` did not work because electron does not support electron binaries.

To fix this, I installed electron from the *tur-repo*, and then ran `ELECTRON_SKIP_BINARY_DOWNLOAD=1 pnpm i`, and this seemed to have worked.

However, upon running `pnpm start`, the entire electron broke and it couldn't get the paths. This is because if you go to node_modules/electron/index.js, it complains about a missing path.txt. Create a path.txt in node_modules/electron, fill it with anything (doesn't matter because you will populate the use of this with something else).

Then, go to cli.js. There is a line that says `proc.spawn(electron, process...)`. Replace electron with 'electron'. Return back to the home directory.

Now, running pnpm start seems to run without errors. However, this hanged my terminal, eventually crashing my Termux from memory usage. I am not quite sure what the problem was, but for some reason, the package.json's start script seemed to have caused this. Run `pnpm build` and `electron .`, and you will have a working Vesktop.

## Why partially working?
Unfortunately, `electron .` seems to be a rather fragile way to run Vesktop, and probably not a distributable way either. 

However, `pnpm build` and then a custom command for `electron-builder --linux tar.gz` seemed to not have worked. There is an obsecure error with networking and resolution trying to download Electron v29.1.1. This looks like it'll be pretty nasty to patch, so maybe I'll look into this later. For now, the environment isn't *too* unstable, so I guess it will work.
