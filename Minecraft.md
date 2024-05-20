# Minecraft Launcher and other launchers
Status: Not working.

## What I've tried
Standard Minecraft launcher, did not work. Majority of closed-source launchers will not work. The reason is because they ship their own version of java runtime, which is an x64 java, and will not run on ARM devices. This may not be fixed through Box64, because the java is called from inside code so it will ignore box64.
