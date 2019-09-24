---
layout: post
title: Howto package from local git with osc and obs-service-obs_scm
category: documentation
---

We will show you a method to work with 
osc/obs-service-obs_scm mostly local, which is not commonly known.

## Preamble

The goal is to shorten the roundtrip from changing your code to the testing of
a built package  without having to commit/push to OBS or your git repo over
and over again while development. 

Another benefit is that you can test your entire project as a package on your 
local machine.

## Approach

As an example I will use the ```obs-testpackage```, which can be found here:

* OBS: https://build.opensuse.org/package/show/home:M0ses/obs-testpackage
* GitHub: https://github.com/M0ses/obs-testpackage

As a best practice I recommend to commit the spec file inside the ```dist/```
directory, this way your packaging information is always aligned to the 
corresponding code stream. Precondition for the following steps are a project
and a package inside OBS. For this example I will use

* Project: home:foo
* Package: obs-testpackage

### 1. Checkout your package from OBS locally

```
osc co home:foo/obs-testpackage
```

### 2. Change into the directory

```
cd home:foo/obs-testpackage
```

### 3. Create a ```_service``` file inside home:foo/obs-testpackage

Edit ```_service```:

```
<services>
  <service name="obs_scm">
    <param name="url">https://github.com/M0ses/obs-testpackage</param>
    <param name="scm">git</param>
    <param name="versionformat">@PARENT_TAG@</param>
    <param name="extract">dist/obs-testpackage.spec</param>
  </service>
  <service name="set_version" mode="buildtime"/>
  <service name="tar" mode="buildtime"/>
  <service name="recompress" mode="buildtime">
    <param name="compression">gz</param>
    <param name="file">*.tar</param>
  </service>
</services>
```

and commit it to OBS

```
osc add _service
osc ci
```


### 4. Run service the first time

```
osc service lr
```

Afterwards you find a directory in your current package directory which
contains a working copy of your git repo

### 5. change into your working copy

```
cd obs-testpackage
# or
cd _servicecache:obs-testpackage
```

### 6. create a local branch

```
git checkout -b local_branch

```

### 7. go back to your OBS package directory


```
cd -
```

### 8. Add the local branch information in the xml inside ```_service``` file under ```services -> service obs_scm -> param revision```

```
<services>
  <service name="obs_scm">
    ...
    <param name="revision">local_branch</param>
  </service>
  ...
</services>
```

### 9. Go back to your git working copy

```
cd -
```

### 10. Make your changes/Edit your code

### 11. Commit your changes to ```local_branch```

```
git add .
git ci -m "my commit message"
```

### 12. (Optional) tag your branch to get a new version


```
git tag <my.new.tag>
```


### 13. Change directory to your package directory


```
cd -
```

### 14. build your package with ```osc```

```
osc build
```

### 15. Install your packages

The last output of the ```obs build``` command looks like this

```
[    4s] 
[    4s] 
[    4s] fs-x270 finished "build obs-testpackage.spec" at Fri Sep 20 08:25:11 UTC 2019.
[    4s] 

/var/tmp/build-root/openSUSE_Tumbleweed-x86_64/home/abuild/rpmbuild/SRPMS/obs-testpackage-0.0.7-0.src.rpm
/var/tmp/build-root/openSUSE_Tumbleweed-x86_64/home/abuild/rpmbuild/RPMS/x86_64/obs-testpackage-0.0.7-0.x86_64.rpm
```

Now you can install the rpm packages via


```
rpm -Uhv --force </path/to/rpm/package.rpm>
```


### 16. Enjoy!

Now you can repeat from step 9 until your are done with your changes.

### 17. Finalize your changes

#### 17.1 git

Now you can push your changes to your git repo or create a pull request.
If your git repo has an trigger mechanism for OBS building the new package should be started 
automatically. But thats another Story.

#### 17.2 osc/OBS

To cleanup your local OBS working copy, you could do a (inside home:foo/obs-testpackge)

```
rm -rf *
osc up
```

to get rid of the 

```
<param name="revision">local_branch</param>
```

in your local OBS project/package directory


