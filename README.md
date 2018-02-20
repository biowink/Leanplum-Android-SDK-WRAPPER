##### Latest version:
[![](https://jitpack.io/v/biowink/Leanplum-Android-SDK-WRAPPER.svg?style=flat-square)](https://jitpack.io/#biowink/Leanplum-Android-SDK-WRAPPER)

This repository wraps the [official Leanplum SDK git repository][Leanplum SDK repo], allowing to build it from [JitPack] instead of relying on the provided binaries, which are affected by a few issues:

+ no documentation
+ no sources
+ some parts are obfuscated
+ weird ProGuard setup
+ slow to accept external PR

It's embedded in this repository as a [git submodule] and never modified directly, which presents a few challenges:

+ #### Gradle build not configured properly
  The main clean/assemble/publish command sequence fails:
  ```bash
  ./gradlew clean assembleRelease publishAarPublicationToMavenLocal
  ```
  because the custom publishing tasks don't declare dependencies on the assembling tasks

+ #### Non-Gradle entry point
  Given the previous point, a [bash script] is used to run the Gradle commands one by one

Gradle provides two ways to wrap an existing build:

+ [Composite builds]: only allows to replace dependency in the wrapped build
+ [Multi-project builds]: can only have one settings file (`settings.gradle`) and one root build file (`build.gradle`)

Since none of them directly provide what's needed, the solution implemented here is a multi-project build where [`settings.gradle`] calls the _wrapped_ `settings.gradle` and delegates the role of "root project" to the wrapped project. Then, instead of calling _directly_ the wrapped `build.gradle`, it first [creates a copy][merge] of it and appends some [custom logic] that fixes the tasks dependencies and adds the sources jar to the published aar.

### Notes

+ Building on JitPack is customised in [`jitpack.yaml`], see the [documentation][JitPack docs] for more information.

+ The final "artifact version" is the result of [`git describe`], so the git tag for a version should ideally mirror the relative version on Leanplum (at the time of writing this, it's `4.0.0`).

+ While we wait for [our changes][StreamReference PR] to be merged upstream, the git submodule currently points to [our fork][Leanplum SDK fork].

+ To update the commit the submodule points to:
  ```bash
  # from the root project, go into submodule directory
  cd 'Leanplum-Android-SDK'
  
  # fetch latest changes
  git fetch
  
  # check out new tag/commit
  git checkout '6.6.6'
  
  # go back to root project
  cd -
  
  # add changed submodule reference
  git add 'Leanplum-Android-SDK'
  
  # create commit
  git commit
  
  # create tag
  git tag '6.6.6'
  
  # push commit
  git push
  
  # push tag
  git push --tags
  ```

[bash script]: https://github.com/Leanplum/Leanplum-Android-SDK/blob/4.0.0/build.sh
[`jitpack.yaml`]: /jitpack.yml
[`settings.gradle`]: /settings.gradle
[custom logic]: /build.gradle
[merge]: /util/merge-build-file.gradle

[Leanplum SDK repo]: https://github.com/Leanplum/Leanplum-Android-SDK
[Leanplum SDK fork]: https://github.com/biowink/Leanplum-Android-SDK/tree/feature/stream-reference
[StreamReference PR]: https://github.com/Leanplum/Leanplum-Android-SDK/pull/157

[JitPack]: https://jitpack.io/#biowink/Leanplum-Android-SDK-WRAPPER
[JitPack docs]: https://jitpack.io/docs/

[git submodule]: https://git-scm.com/docs/gitsubmodules
[`git describe`]: https://git-scm.com/docs/git-describe

[Composite builds]: https://docs.gradle.org/current/userguide/composite_builds.html
[Multi-project builds]: https://docs.gradle.org/current/userguide/intro_multi_project_builds.html
