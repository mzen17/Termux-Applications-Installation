# Vesktop [Discord]
Status: WORKING!!! (See update).
![vesktop-working](https://github.com/mzen17/Termux-Applications-Installation/assets/92705460/9c02298c-d685-4f4d-8384-c22b5f6a28b9)


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

## UPDATE!!!! [5/29/2024]
I was able to get vesktop working (electron-builder)!!!!!!!!!!!

After many long hours of painful debugging, I found a workaround to get vesktop to work. It turns out that by forcing the app-builder that electron-builder uses ([link here](https://github.com/develar/app-builder)) to copy native system electron instead of bringing its own, Vesktop works and is compiled without needing glibc libraries. The glibc version did not work, mostly because of the fact that ffmpeg.so has not been made into a termux library yet (for some reason manual compilation of that library didn't work). 

This is the primary patch that I used:
[app-builder/pkg/electron/electronUnpack.go]

```
package electron

import (
//	"os/exec"
	"path/filepath"
	"fmt"
	"io"
	"os"
	"io/ioutil"
	"github.com/alecthomas/kingpin"
	"github.com/develar/app-builder/pkg/util"
)

func ConfigureUnpackCommand(app *kingpin.Application) {
	command := app.Command("unpack-electron", "")
	jsonConfig := command.Flag("configuration", "").Short('c').Required().String()
	outputDir := command.Flag("output", "").Required().String()
	distMacOsAppName := command.Flag("distMacOsAppName", "").Default("Electron.app").String()

	command.Action(func(context *kingpin.ParseContext) error {
		var configs []ElectronDownloadOptions
		err := util.DecodeBase64IfNeeded(*jsonConfig, &configs)
		if err != nil {
			return err
		}
		return UnpackElectron(configs, *outputDir, *distMacOsAppName, true)
	})
}

func copyFile(src, dest string) error {
	// Open the source file
	srcFile, err := os.Open(src)
	if err != nil {
		return err
	}
	defer srcFile.Close()

	// Create the destination file
	destFile, err := os.Create(dest)
	if err != nil {
		return err
	}
	defer destFile.Close()

	// Copy the contents from the source file to the destination file
	_, err = io.Copy(destFile, srcFile)
	if err != nil {
		return err
	}

	// Ensure the destination file is properly saved
	err = destFile.Sync()
	if err != nil {
		return err
	}

	return nil
}

func UnpackElectron(configs []ElectronDownloadOptions, outputDir string, distMacOsAppName string, isReDownloadOnFileReadError bool) error {
	fmt.Println("THESTUFF: " + outputDir)
	
	var fpath = "/data/data/com.termux/files/usr/lib/electron"

	files, err := ioutil.ReadDir(fpath)
	if err == nil {}

	// Copy each file to the destination directory
	for _, file := range files {
		if !file.IsDir() { // Only process files, not subdirectories
			srcFile := filepath.Join(fpath, file.Name())
			destFile := filepath.Join(outputDir, file.Name())
			
			err := copyFile(srcFile, destFile)
			if err != nil {
				fmt.Println("Error copying file:", err)
				return nil
			}
		}
	}

	fmt.Println("Files successfully copied!")

	if len(distMacOsAppName) == 0 {
		distMacOsAppName = "Electron.app"
	}

	excludedFiles := make(map[string]bool)
	excludedFiles[filepath.Join(outputDir, distMacOsAppName, "Contents", "Resources", "default_app.asar")] = true
	excludedFiles[filepath.Join(outputDir, "resources", "default_app.asar")] = true

	excludedFiles[filepath.Join(outputDir, "resources", "inspector", ".htaccess")] = true

	excludedFiles[filepath.Join(outputDir, "version")] = true
	fmt.Println(excludedFiles)

	return nil
}
```
It's really dirty, and the "excludedFiles" got lost, but it still seems to be working. As you can see here, glibc-check (which finds if a binary was compiled with GCC, script taken from stackoverflow) reports nothing.
![glibc-check](https://github.com/mzen17/Termux-Applications-Installation/assets/92705460/1b332a9b-ec61-4723-b9b0-3ff7b6d87780)

There still seems to be some GPU bugs (I use the mesa freedreno dri3 driver), but it seems to be the driver not Vesktop.
![vesktop-bugs](https://github.com/mzen17/Termux-Applications-Installation/assets/92705460/d22eda67-7d7c-4e57-8c13-7cf6c8b2bd5e)

Anyways, I'll be dropping the tarball in a cdn ([click to download](https://repository.termux.mzen.dev/vesktop-1.5.2-arm64-termux.tar.gz)) if anyone wants to download it.
