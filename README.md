# BattleBitAPIRunner

Modular battlebit community server api runner. Lets you host community servers with modules without having to code.

## Features

- Start the community server API endpoint for your server to connect to
- Modules are simple C# source files `.cs`
- Modules support module dependencies
- Modules support optional module dependencies
- Modules support binary dependencies (Newtonsoft.Json, System.Net.Http, ...)
- Integrated Per-module and per-server configuration files

## Configuration
Configure the runner in the `appsettings.json`:
```
{
  "IP": "127.0.0.1",
  "Port": 29595,
  "ModulePath": "./modules",
  "Modules": [ "C:\\path\\to\\specific\\ModuleFile.cs" ],
  "DependencyPath": "./dependencies",
  "ConfigurationPath": "./configurations"
}
```
- IP: Listening IP
- Port: Listening port
- ModulePath: Path to the folder containing all modules (is created if not exist)
- Modules: Array of individual module file paths
- DependencyPath: Path to the folder containing all binary (dll) dependencies (is created if not exist)
- ConfigurationPath: Path to the folder containing all module and per-server module configuration files (is created if not exist)

Module and per-server module configurations are located in the configurations subdirectory, if you have not changed the path.

## Usage

Download the latest release, unpack, configure and start `BattleBitAPIRunner`.
The modules, dependencies and configurations folder will be created in the same directory as the executable, if you have not specified a different path in the configuration file.
Modules are loaded upon startup. To reload modules the application has to be restarted.

Place modules in the modules folder or specify their path in the configuration file.
Place binary dependencies in the dependencies folder.

## Developing modules

Modules are .net 6.0 C# source code files. They are compiled in runtime when the application starts.
To debug a module, simply attach your debugger to the BattleBitAPIRunner process.

To create a module, create a (library) .net 6.0 C# project.
Set `ImplicitUsings` to `disabled`, for example by unchecking `Enable implicit global usings to be declared by the project SDK.` in the project settings.
Add a nuget dependency to [BBRAPIModules](https://www.nuget.org/packages/BBRAPIModules).
In your module source file, have exactly one public class which has the same name as your file and inherit `BBRAPIModules.BattleBitModule`.
Your module class now has all methods of the BattleBit API, such as `OnConnected`.

### Optional module dependencies
To optionally use specific modules, add a public property of type `BattleBitModule` or `dynamic` to your module and add the `[ModuleReference]` attribute to it. Make sure the name of the property is the name of the required module.
```cs
[ModuleReference]
public BattleBitModule? PlayerFinder { get; set; }
// or using dynamic
[ModuleReference]
public dynamic? RichText { get; set; }
```
When all modules are loaded (`OnModulesLoaded`) the dependant module will be available on this property, if it was loaded.

You can call methods on that module by using the `Call` method or by invoking the method dynamically.

```cs
this.PlayerFinder?.Call("TargetMethod");
this.PlayerFinder?.Call("TargetMethodWithParams", "param1", 2, 3);
bool? result = this.PlayerFinder?.Call<bool>("TargetMethodWithReturnValue");
// or using dynamic
this.RichText?.TargetMethod();
this.RichText?.TargetMethodWithParams("param1", 2, 3);
bool? result = this.RichText?.TargetMethodWithReturnValue();
```

### Required module dependencies
To require a dependency to another module, include the required module source file in your project (optional, only for syntax validation and autocomplete).
Add a `[RequireModule(typeof(YourModuleDependency))]` attribute to your module class. Multiple attributes for multiple required dependencies are supported.

```cs
[RequireModule(typeof(PlayerPermissions))]
[RequireModule(typeof(CommandHandler))]
public class MyModule : BattleBitModule
```

You will also have to add the module properties to your class as you would do with optional module dependencies, except they will be guaranteed to not be null after `OnModulesLoaded`.

### Module Configuration
Create a class containing public properties of all your configuration variables and inherit from `ModuleConfiguration`.
Add a public property of your configuration class to your module.
If the property is static it will be a global configuration shared by all instances of your module across all servers.
If the property is not static, it will be a per-server configuration.
You can have multiple configurations, static and non-static, per module.
The configuration file will be called like the property name.

```cs
public class MyModuleConfiguration : ModuleConfiguration
{
    public string SomeConfigurationValue { get; set; } = string.Empty;
}
public class MyModule : BattleBitModule
{
    public MyModuleConfiguration GlobalConfig { get; set; }
    public MyModuleConfiguration PerServerConfig { get; set; }
}
```
This will create a `./configurations/MyModule/GlobalConfig.json` and a `./configurations/127.0.0.1_29595/MyModule/PerServerConfig.json` (for each server) configuration file.

# Modules
## Example modules
- https://github.com/RainOrigami/BattleBitExamples

## Base modules
- https://github.com/RainOrigami/BattleBitBaseModules - A collection of basic modules to get you started (MOTD, PlayerFinder, PlayerPermissions, CommandHandler, PermissionsCommands, DiscordWebhooks)

## Other modules
- https://github.com/RainOrigami/BattleBitZombies - 28 days later inspired zombie pvp game mode module

# Features to come
- You can suggest some over at the [Blood is Good Discord](https://discord.bloodisgood.org)
