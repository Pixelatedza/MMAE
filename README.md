# Motorsport Manager Assembly Edits

**Remember to backup any files you change**

The following will show you what to change in the *Assembly-CSharp.dll* file. The file can be easily edited using [dnSpy](https://github.com/0xd4d/dnSpy). You can just scroll down to the releases section, it will lead you to the download page.

When your at the place you want to change the code, simply right click and say edit method (C#) or edit class (C#). They will both open an editor where you can make the changes. Once the changes are made, click the compile button and hope everything compiles happily.

## Perfect Setup
SessionSetup -> GetSetupOutput(ref SessionSetup.SetupOutput outSetupOutput)

```C#
if (this.mVehicle.isPlayerDriver)
{
    this.mAISetupOutput = new SessionSetup.SetupOutput();
    this.SetAISetupWithinRange(0.99f, 1f);
    outSetupOutput.aerodynamics = this.mAISetupOutput.aerodynamics;
    outSetupOutput.speedBalance = this.mAISetupOutput.speedBalance;
    outSetupOutput.handling = this.mAISetupOutput.handling;
    return;
}
```

## AI Controlled Pitstops
ConsoleCommands -> useAIForDrivers

```C#
public static bool useAIForDrivers = true;
```
