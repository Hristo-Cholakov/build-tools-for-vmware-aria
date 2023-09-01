## Breaking Changes


## Deprecations


## Features


## Improvements

### Fixed the compiled SAGA workflow crashes when no imports are defined in saga yaml

#### Previous Behaviour

When no imports were defined in typescript SAGA workflow file format, the compiled vRO workflow failed when being executied in vRO on Initilize scriptable task
with null object error.

#### New Behaviour

When no imports are defined in typescript SAGA workflow file format, the compiled vRO workflow runs successfully.

### Fixed backup of vRO packages so that the highest available version is backed up
#### Previous Behavior

Back up of vRO packages (using the flag in the environment.properties file: vro_enable_backup=true)
would only work if the currently imported packages (which are to back up), had the same version as the one in vRO.
Otherwise, the import would throw an '404 Not found' exception and break the import process,
due to not finding the same package and version to back up.

#### New Behavior
Back up of vRO packages now works by:
- backing up the highest available version in vRO of the imported package,
- logging a message that back up is skipped for the package, if no versions of it are found in vRO, continuing with backup of next packages, and the import process.

## Upgrade procedure
