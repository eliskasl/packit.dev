---
title: "Initial version (0.1.0) of packit is out!"
date: 2019-03-08T15:44:36+01:00
draft: false
---

We would like to announce general availability of the initial version of
packit, titled '0.1.0'.

Since this is our first release, we would like to ask you to be patient if you
encounter any issues. We work hard on packit's usability. If you feel like that
packit is doing something weird or if anything is unclear, don't hesitate and
reach out to us by creating [a new Github
issue](https://github.com/packit-service/packit/issues/new).

The initial release contains two commands:

* `packit propose-update` — Opens a pull request in dist-git for the latest
  upstream release of a selected repository.
* `packit watch-releases` — Watches events for all the upstream releases and
  performs `propose-update` for those who use packit.
<!--more-->

## Installation

```
$ dnf install --enablerepo=updates-testing packit
```

Or

```
$ pip3 install --user packitos
```

Or (if you're brave)

```
$ pip3 install --user git+https://github.com/packit-service/packit
```


## Requirements

Present features have strict requirements on the upstream projects:

* You need to have a packit config file present in the upstream repo.

* You need to have spec file present in the upstream repo.

This workflow is suitable for people who are both upstream and downstream
maintainers of the particular project. If you don't fit into that bucket, then
packit might not be ready for you, yet. Please wait till we land more
[source-git](https://github.com/packit-service/packit/blob/master/docs/source-git.md)
related functionality into packit.


## `propose-update`

I'm going to demonstrate this functionality on
[ogr](https://github.com/packit-service/ogr.git), our library for git forges,
which powers packit.

It was recently approved for Fedora, so we can use packit to bring the initial
version of ogr into Fedora Rawhide, 30 and 29.


### Do we have everything?

Let's see [the upstream
guide](https://github.com/packit-service/packit/blob/master/docs/update.md) for
the `propose-update` command on what we need:


#### 0. The upstream repository with a valid upstream release.

```
$ git remote -v
origin  git@github.com:TomasTomecek/ogr.git (fetch)
origin  git@github.com:TomasTomecek/ogr.git (push)
upstream        https://github.com/packit-service/ogr.git (fetch)
upstream        https://github.com/packit-service/ogr.git (push)
```

Yup.

```
$ git tag --list
0.0.1
0.0.2
0.0.3

$ git checkout 0.0.3
Note: checking out '0.0.3'.
```

And the tag name is matching the version in a spec file:
```
$ grep Version python-ogr.spec
Version:        0.0.3
```

#### 1. Packit config file placed in the upstream repository.

```
$ ll .packit.yaml
-rw-rw-r--. 1 tt tt 177 Mar  1 17:44 .packit.yaml
```

Check.


#### 2. Spec file present in the upstream repository.

```
$ ll python-ogr.spec
-rw-rw-r--. 1 tt tt 1.3K Mar  1 17:43 python-ogr.spec
```

:+1:


#### 3. Pagure API tokens for Fedora Dist-git.

```
$ env | grep TOKEN
PAGURE_USER_TOKEN=will
PAGURE_FORK_TOKEN=not
GITHUB_TOKEN=share, sorry
```

#### 4. Valid Fedora Kerberos ticket.

```
$ kinit ttomecek@FEDORAPROJECT.ORG
Password for ttomecek@FEDORAPROJECT.ORG:

$ klist
Ticket cache: KEYRING:persistent:1024:krb_ccache_g0t1Ty3Ah
Default principal: ttomecek@FEDORAPROJECT.ORG

Valid starting       Expires              Service principal
03/01/2019 18:12:25  03/02/2019 18:12:19  krbtgt/FEDORAPROJECT.ORG@FEDORAPROJECT.ORG
        renew until 03/08/2019 18:12:19
```

We're all set!


### Time to shine

We are still in the "ogr" upstream git repository.

```
$ packit propose-update
INFO: Running 'anitya' versioneer
ERROR: Failed to determine latest upstream version!
Check that the package exists on https://release-monitoring.org.
using "master" dist-git branch
syncing ./python-ogr.spec
INFO: Downloading file from URL https://files.pythonhosted.org/packages/source/o/ogr/ogr-0.0.3.tar.gz
100%[=============================>]    17.95K  eta 00:00:00
downloaded archive: /tmp/tmp2e65b0xt/ogr-0.0.3.tar.gz
uploading to the lookaside cache
PR created: https://src.fedoraproject.org/rpms/python-ogr/pull-request/1
```

Mind-blowing, isn't it? Now we have latest python-ogr in Fedora Rawhide by
running only a single command.

I have also [added](https://release-monitoring.org/project/18832/) ogr into release-monitoring as packit suggests.

Once we are okay with the changes, we have to [merge the pull
request](https://src.fedoraproject.org/rpms/python-ogr/pull-request/1). That's
our responsibility, as maintainers.


### Building in koji

Time to build the package (packit doesn't support building in koji, yet)

```
$ fedpkg clone python-ogr
Cloning into 'python-ogr'...
remote: Counting objects: 8, done.
remote: Compressing objects: 100% (5/5), done.
remote: Total 8 (delta 0), reused 5 (delta 0)
Receiving objects: 100% (8/8), done.

$ cd python-ogr

$ git log
commit c298df5e540ba1d010366e102c1c75d4f5b0b0cc (HEAD -> master, origin/master, origin/HEAD)
Author: Tomas Tomecek <ttomecek@redhat.com>
Date:   Fri Mar 1 18:15:00 2019 +0100

    [packit] 0.0.3 upstream release

    more info

    Signed-off-by: Tomas Tomecek <ttomecek@redhat.com>

commit 7d5ab1471ca0ee2a6c0254410b83beaa83b80f0b
Author: Gwyn Ciesla <limb@fedoraproject.org>
Date:   Fri Mar 1 15:18:34 2019 +0000

    Added the README
```

Yup, that's our commit. `more info` was added there by accident, this is
already fixed in packit.

```
$ fedpkg build
Building python-ogr-0.0.3-1.fc31 for rawhide
Created task: 33125435
Task info: https://koji.fedoraproject.org/koji/taskinfo?taskID=33125435
Watching tasks (this may be safely interrupted)...
33125435 build (rawhide, /rpms/python-ogr.git:c298df5e540ba1d010366e102c1c75d4f5b0b0cc): free
33125435 build (rawhide, /rpms/python-ogr.git:c298df5e540ba1d010366e102c1c75d4f5b0b0cc): free -> open (buildvm-14.phx2.fedoraproject.org)
  33125451 buildArch (python-ogr-0.0.3-1.fc31.src.rpm, noarch): open (buildvm-14.phx2.fedoraproject.org)
  33125436 buildSRPMFromSCM (/rpms/python-ogr.git:c298df5e540ba1d010366e102c1c75d4f5b0b0cc): closed
33125435 build (rawhide, /rpms/python-ogr.git:c298df5e540ba1d010366e102c1c75d4f5b0b0cc): open (buildvm-14.phx2.fedoraproject.org) -> closed
  0 free  1 open  2 done  0 failed
  33125464 tagBuild (noarch): closed
  33125451 buildArch (python-ogr-0.0.3-1.fc31.src.rpm, noarch): open (buildvm-14.phx2.fedoraproject.org) -> closed
  0 free  0 open  4 done  0 failed

33125435 build (rawhide, /rpms/python-ogr.git:c298df5e540ba1d010366e102c1c75d4f5b0b0cc) completed successfully
```

That was rough, can't wait to do this with packit.


Let's do Fedora 30 now:

```
$ packit propose-update --dist-git-branch f30
INFO: Running 'anitya' versioneer
using "f30" dist-git branch
syncing ./python-ogr.spec
INFO: Downloading file from URL https://files.pythonhosted.org/packages/source/o/ogr/ogr-0.0.3.tar.gz
100%[=============================>]    17.95K  eta 00:00:00
downloaded archive: /tmp/tmpl5xxq22x/ogr-0.0.3.tar.gz
uploading to the lookaside cache
PR created: https://src.fedoraproject.org/rpms/python-ogr/pull-request/3
```

And so on...


## Conclussion

As you can see, packit is useful for us right away.

We'll be delighted if you try it out and let us know what you think.
