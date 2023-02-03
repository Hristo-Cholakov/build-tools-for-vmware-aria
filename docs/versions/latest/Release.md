[//]: # (VERSION_PLACEHOLDER DO NOT DELETE)
[//]: # (Used when working on a new release. Placed together with the Version.md)
[//]: # (Nothing here is optional. If a step must not be performed, it must be said so)
[//]: # (Do not fill the version, it will be done automatically)
[//]: # (Quick Intro to what is the focus of this release)

## Breaking Changes
[//]: # (### *Breaking Change*)
[//]: # (Describe the breaking change AND explain how to resolve it)
[//]: # (You can utilize internal links /e.g. link to the upgrade procedure, link to the improvement|deprecation that introduced this/)



## Deprecations
[//]: # (### *Deprecation*)
[//]: # (Explain what is deprecated and suggest alternatives)



[//]: # (Features -> New Functionality)
## Features
[//]: # (### *Feature Name*)
[//]: # (Describe the feature)
[//]: # (Optional But higlhy recommended Specify *NONE* if missing)
[//]: # (#### Relevant Documentation:)



[//]: # (Improvements -> Bugfixes/hotfixes or general improvements)
## Improvements
[//]: # (### *Improvement Name* )
[//]: # (Talk ONLY regarding the improvement)
[//]: # (Optional But higlhy recommended)
[//]: # (#### Previous Behavior)
[//]: # (Explain how it used to behave, regarding to the change)
[//]: # (Optional But higlhy recommended)
[//]: # (#### New Behavior)
[//]: # (Explain how it behaves now, regarding to the change)
[//]: # (Optional But higlhy recommended Specify *NONE* if missing)
[//]: # (#### Relevant Documentation:)

### Enabled Unit testing for npm lib based projects

#### Previous Behavior

npm test was not part of maven lifecycle for npm based projects

#### New Behavior

npm test is now triggered during maven package goal for npm based projects

#### Relevant Documentation

None

### Fix polyglotpkg/abx backward compatibility

#### Previous Behavior

If you upgrade to latest Build Tools for VMware Aria version the previously created ABX and polyglot actions will stop working

#### New Behavior

With small adjustments, the old ABX and polyglot actions should be upgradable

#### Relevant Documentation

None

### Fix polyglotpkg dependencies

#### Previous Behavior

If you build project with polyglot + typescript-project-all the build will fail with dependency resolution error

#### New Behavior

If you build project with polyglot + typescript-project-all the build will succeed

#### Relevant Documentation

None

### Fix error Command line is too long

#### Previous Behavior

If you build the project with lot of dependencies which exceeds the command line text limitation of 8191 the build will fail with Command line is too long error

#### New Behavior

If you build project with any number of dependencies the build will succeed

#### Relevant Documentation

None

## Upgrade procedure:
[//]: # (Explain in details if something needs to be done)

[//]: # (## Changelog:)
[//]: # (Pull request links)
