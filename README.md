# Motorsport Manager Assembly Edits

**Remember to backup any files you change**

The following will show you what to change in the *Assembly-CSharp.dll* file. The file can be edited using [dnSpy](https://github.com/0xd4d/dnSpy). You can just scroll down to the releases section, it will lead you to the download page.

> Note! You need the other DLLs from the *Assembly-CSharp.dll* folder to compile the edited code.

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

## Macro Management Mod
ConsoleCommands -> useAIForDrivers

```C#
public static bool useAIForDrivers = true;
```

# Below thanks to shortshifted78 and others
## No Micromanagement mod in SessionStrategy
```C#
private void UpdateDrivingStyleAndEngineModes()
{
    bool arg_06_0 = this.mUsesAIForStrategy;
    if (this.mVehicle.pathState.IsInPitlaneArea() || this.mVehicle.timer.hasSeenChequeredFlag)
    {
        this.mVehicle.performance.drivingStyle.SetDrivingStyle(global::DrivingStyle.Mode.BackUp);
        this.mVehicle.performance.fuel.SetEngineMode(global::Fuel.EngineMode.Low, false);
    }
    else
    {
        this.mVehicle.performance.drivingStyle.SetRacingAIDrivingStyle();
        this.mVehicle.performance.fuel.SetRacingAIEngineMode();
    }
    if (global::Game.instance.sessionManager.isSafetyCarFlag)
    {
        this.mVehicle.performance.drivingStyle.SetDrivingStyle(global::DrivingStyle.Mode.BackUp);
        this.mVehicle.performance.fuel.SetEngineMode(global::Fuel.EngineMode.Low, false);
    }
    if (this.mVehicle.car.seriesCurrentParts[1].partCondition.IsOnRed())
    {
        this.mVehicle.performance.fuel.SetEngineMode(global::Fuel.EngineMode.Low, false);
    }
}
```

## Automatic Low Fuel and Tire Saving Mode
This one is good for those that are playing with full human control of your drivers. It will instantly kick your drivers into fuel and tire saving mode as soon as they hit pit lane or if a safety car is called out. The down side is that after pitting or when the yellow ends, they will go to semi random push and engine levels.

SessionStrategy -> UpdateDrivingStyleAndEngineModes()

```C#
private void UpdateDrivingStyleAndEngineModes()
{
    this.mUsesAIForStrategy = this.mVehicle.driver.personalityTraitController.UsesAIForStrategy(this.mVehicle);
    if (this.mUsesAIForStrategy || !this.mVehicle.isPlayerDriver || Game.instance.sessionManager.isUsingAIForPlayerDrivers)
    {
        if (this.mVehicle.pathState.IsInPitlaneArea() || this.mVehicle.timer.hasSeenChequeredFlag)
        {
            this.mVehicle.performance.drivingStyle.SetDrivingStyle(DrivingStyle.Mode.BackUp);
            this.mVehicle.performance.fuel.SetEngineMode(Fuel.EngineMode.Low, false);
        }
        else
        {
            this.mVehicle.performance.drivingStyle.SetRacingAIDrivingStyle();
            this.mVehicle.performance.fuel.SetRacingAIEngineMode();
        }
        if (Game.instance.sessionManager.isSafetyCarFlag)
        {
            this.mVehicle.performance.drivingStyle.SetDrivingStyle(DrivingStyle.Mode.BackUp);
            this.mVehicle.performance.fuel.SetEngineMode(Fuel.EngineMode.Low, false);
        }
        if (this.mVehicle.car.seriesCurrentParts[1].partCondition.IsOnRed())
        {
            this.mVehicle.performance.fuel.SetEngineMode(Fuel.EngineMode.Low, false);
        }
    }
    else
    {
        if (this.mVehicle.pathState.IsInPitlaneArea() || this.mVehicle.timer.hasSeenChequeredFlag)
        {
            this.mVehicle.performance.drivingStyle.SetDrivingStyle(DrivingStyle.Mode.BackUp);
            this.mVehicle.performance.fuel.SetEngineMode(Fuel.EngineMode.Low, false);
        }
        if (this.mVehicle.pathController.currentPathType == PathController.PathType.PitlaneExit)
                {
                    PathController.Path path = this.mVehicle.pathController.GetPath(PathController.PathType.PitlaneExit);
                    if (path.data.gates.Count - (path.nextGateIdAccessor + 1) < 5)
                    {
                            this.mVehicle.performance.drivingStyle.SetRacingAIDrivingStyle();
                            this.mVehicle.performance.fuel.SetRacingAIEngineMode();
                    }
                }
        if (Game.instance.sessionManager.isSafetyCarFlag)
        {
            this.mVehicle.performance.fuel.SetEngineMode(Fuel.EngineMode.Low, false);
        }
    }
}
```

## Season Breakthroughs
This one will help control any start of the season breakthroughs. You will need to find these lines in the code and adjust. If you have FlamingRed's mod (or shortshifted78's), there will be two sets of these numbers, and you should adjust both

CarPartDesign -> SetSeasonStartingStats()

```C#
IsPlayersTeam - random < 0.01
!this.mTeam.IsPlayersTeam - (minor) random < 0.04
!this.mTeam.IsPlayersTeam - (major) random < 0.02
```

## AI Minimum Stripdown
This next one should be for the AI to determine the minimum they will strip their car down to

CarPartStats
```C#
weightStrippedReliabilityMin = 0.6f
```

## Saftey Car Count
Crashes! this is a big one. Now if you are playing with a series that doesn't have long races, you are welcome to leave the safety car count alone. I, (shortshifted78), bumped my numbers up because of the endurance series. I also tried to look at historical data for crashes in the endurance series. With the numbers below you are guaranteed to have 1 crash at some point in the race, and heavy rain should add at least 1 more if the game has decided on a number that results in fewer than 4 crashes.

CrashDirector -> OnSessionStarting()
```C#
case PrefGameRaceLength.Type.Short:
    this.mRealSafetyCarCount = 2;
case PrefGameRaceLength.Type.Medium:
    this.mRealSafetyCarCount = 4;
if (random > 0.98f)
    i = 6;
else if (random > 0.95f)
    i = 5;
else if (random > 0.90f)
    i = 4;
else if (random > 0.50f)
    i = 3;
else if (random > 0.10f)
    i = 2;
else
    i = 1;
if (this.mSessionManager.currentSessionWeather.GetAverageWeather().rainType >= Weather.RainType.Heavy && i < 4)
    i += RandomUtility.GetRandom(1, 3);
```

## Corner Cutting Fix
Now what about cutting corners? in the coding? no! the cars on track! Apparently the length of the season dictates the number of corner cuts you can have in a season. So since the lower tiers have fewer races (F3 has just 8 races) and are supposed to be populated with young drivers moving up in the ranks, they might be more prone to cutting corners due to inexperience. So for the shortest season 20 corner cuts max. Since my endurance series is in the 9-10 race range, but the races are long, there is a decent chance just due to the sheer number of laps in a race. As the tiers go up, the seasons get longer and in theory the drivers get better so the cuts per race go down.

CutCornerSeasonDirector -> OnSeasonStart()

```C#
int inMax = 20;
if (inChampionship.calendar.Count >= 16)
{
    inMax = 16;
}
else if (inChampionship.calendar.Count >= 12)
{
    inMax = 18;
}
else if (inChampionship.calendar.Count >= 9)
{
    inMax = 22;
}
int i = RandomUtility.GetRandomInc(5, inMax);
```

To finish off the corner cuts, and when they can happen in the race...

```C#
float t = 1f - inVehicle.driver.GetDriverStats().focus / 20f;
float num = Mathf.Lerp(0.9f, 2f, t);
SessionWeatherDetails currentSessionWeather = Game.instance.sessionManager.currentSessionWeather;
bool flag = Game.instance.sessionManager.flag == SessionManager.Flag.Green;
bool flag2 = currentSessionWeather.GetNormalizedTrackWater() > 0.4f && currentSessionWeather.GetNormalizedRain() > 0.2f;
bool flag3 = inVehicle.timer.gapToAhead > 0.1f && inVehicle.timer.gapToAhead < num;
bool flag4 = (inVehicle.timer.gapToBehind > 0.1f && inVehicle.timer.gapToBehind < num) || flag3;
return flag && (flag4 || flag2);
```

## Regen Drivers
To limit them in their marketability I have this to limit max starting marketability to 50

DriverManager -> GenerateRandomStatsData()

```C#
driverStats.marketability = ((inType != RegenManager.RegenType.Replacement) ? ((float)RandomUtility.GetRandom(0, inEntry.GetIntValue("Marketability")) / 200f
```

## Higher Spec Part Performance and Fewer Tweets
GameStatsConstants
```C#
averageCharactersReadPerSecond = 18f
newSeasonMaxPartCap = 550
partResetMaxBonus = 200f
initialMaxReliabilityValue = 0.63f + RandomUtility.GetRandom(0f, 0.12f)
specPartValues = new int[] //Set to be be 40percent of series reset value
    1000, //F1 or WMC
    600,  //F2 or APS
    240,  //F3 or ERS
    680,  //tier 1 GT
    320,  //tier 2 GT
    800,  //WEC LMP1
    480,  //WEC LMP2
    100,  //who knows
    100
daysRecoveredFromSittingOut = 5
scrutineeringChance = 10f
maxConditionTimeToFix = 144f //this is 6 days but teams should not take this long
liveryEditCost = 50000L
lastChampionshipEventWeek = 47
averageMilesPerLap = 2.95  //this value is based on just tracks in my True Endurance mod
frontWingRate = 0.1f;
rearWingRate = 0.2f;
engineRate = 0.8f;
brakesRate = 0.6f;
suspensionRate = 0.6f;
gearBoxRate = 0.7f;
replacementPeopleCount = 25  //if you like having a lot of options for regens as the years go on you can increase this number
minTweetValue = 3
maxTweetValue = 5
```

## Higher Simulation Speeds
The top set of numbers will do that for you.

GameTimer
```C#
public static float[] skipSimSpeed = new float[]
    0f,
    20f,
    40f,
    90f
public static float[] skipSimEventSpeeds = new float[]
    0f,
    1.5f,
    3.25f,
    5.75f
```

## Lockup Fix
If you noticed that lockups only seemed to happen at certain spots of certain tracks its because the coding was set to require a straight of at least 400m (yd?) before having a chance. This meant that lockups could only really happen at 1 or 2 corners at just a handful of tracks. This next line of code opens up many more corners and pretty much every track. The down side is that you will get many lock ups happening early in the race if you retain all the original lockup coding.

PathData -> CalculateLockUpZones()
```C#
float num = 150
```
AILockUpBehaviour ->OnEnter
```C#
this.mLockUpDuration = (float)RandomUtility.GetRandom(1, 3)
```
AILockUpBehaviour ->OnExit
```C#
float random = RandomUtility.GetRandom(0.03f, 0.09f)
```
TyreLockUpDirector IsTyreLockUpViable
```C#
bool flag = Game.instance.sessionManager.flag == SessionManager.Flag.Chequered;
		bool flag2 = inVehicle.timer.gapToAhead < 3.6f - inVehicle.driver.GetDriverStats().braking / 6f;
		bool flag3 = Game.instance.sessionManager.currentSessionWeather.GetNormalizedTrackWater() > inVehicle.driver.GetDriverStats().adaptability / 21f && RandomUtility.GetRandom01() < 0.5f;
		bool flag4 = Game.instance.sessionManager.currentSessionWeather.GetNormalizedTrackRubber() < (20.4f - inVehicle.driver.GetDriverStats().cornering) / 21f && RandomUtility.GetRandom01() < 0.5f;
		bool flag5 = Game.instance.sessionManager.GetNormalizedSessionTime() > inVehicle.driver.GetDriverStats().fitness / 22f;
		return inVehicle.speed <= GameUtility.MilesPerHourToMetersPerSecond(55f) && !isTutorialActiveInCurrentGameState && !inVehicle.behaviourManager.isOutOfRace && !flag && (flag2 & (flag3 || flag4 || flag5)) && inVehicle.sessionEvents.IsReadyTo(SessionEvents.EventType.LockUp);

float mFrequency = 45f
```
