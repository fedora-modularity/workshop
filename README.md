# Modularity workshop

This workshop should teach you how you can create your own modules.

The plan is to utilize existing guides and just link to them.

Before we start, feel free to look into [Stephen Gallagher's](https://github.com/sgallagher) blog posts on module packaging:
 * [Sausage Factory: An introduction to building modules in Fedora](https://sgallagh.wordpress.com/2017/05/30/sausage-factory-an-introduction-to-building-modules-in-fedora/)
 * [Sausage Factory: Advanced module building in Fedora](https://sgallagh.wordpress.com/2017/06/30/sausage-factory-advanced-module-building-in-fedora/)


## Prerequisite

If you plan to attend the workshop, please follow the instructions bellow so
you'll have all the required content ready on your laptop for the workshop.

Install the build tools:

```
$ sudo dnf install module-build-service
```

Clone this repository...

```
$ git clone https://github.com/fedora-modularity/workshop
```

...and switch to `test-module/` directory, initiate a new git repository and
build the test module so all the build dependencies are cached for the workshop
(may be up to 11G (remove `bootstrap: master` line from `test-module.yaml` if
you want to down it to just ~1G):

```
$ cd workshop/test-module
$ git init . && git add . && git commit -m init
$ mbs-build local
```


## High-level agenda

1. [1-minute-intro to modularity](#1-minute-intro)
2. [modulemd](#modulemd)
3. [Workflow](#workflow)
4. [`mbs-build`](#mbs-build)
5. [Let's create autotools module!](#lets-create-autotools-module)
6. [Testing](#testing)
7. [Feedback](#we-need-your-feedback)


# The workshop

TODO:
* [ ] everyone can see my terminal (font and colors are acceptable)
  * don't use black background


## 1-minute-intro

Modularity cuts Linux distributions into **modules**, giving them **independent lifecycles**, and making them available in **multiple streams**. Giving more options of choice to users, and flexibility and control to packagers.

<img src="/img/modularity-basics-1.png" width=600px>

See the **official [Fedora Modularity documentation](https://docs.pagure.org/modularity/)** for more information

## modulemd

Modules are defined in a [modulemd](https://pagure.io/modulemd) file.

As opposed to traditional distributions, where all packages are built and shipped in a single monolythic release, Modularity needs a mechanism of defining which package goes into what module. And that's modulemd.

Have a look at a real example of [nodejs module](https://src.fedoraproject.org/modules/nodejs/blob/master/f/nodejs.yaml).

## Workflow

All modules will use a [platform module](https://src.fedoraproject.org/modules/platform/blob/master/f/platform.yaml) as a build and runtime dependency. Platform is the base system and common, shared userland.

To create a module, you need to define a **top-level package set** representing your module, and **resolve missing dependencies** between your package set and the platform module. A simple example:

<img src="/img/defining-modules-1-simple.png" width=600px>

### More complex module

Unfortunatelly, it's not easy every time. Many packages will have **complex dependencies**:

<img src="/img/defining-modules-2-complex-bad.png" width=600px>

As you might have guessed, bundling all dependencies is not the right thing to do in this example. Instead, we need to identify **other modules**, and use these as dependencies:

<img src="/img/defining-modules-2-complex-good.png" width=600px>

At the **beginning of development of the F27 Server**, all of these modules will need to get indentified and built. However, when we are done with the initial set, packagers will be able to use what's already available.


## Initial coordination

As we saw above, there is a need for an initial coordination while idetifying and defining the initial set of modules. How to handle that?

The [dependency-report](https://github.com/fedora-modularity/dependency-report) repository along with the [modularity-modules](https://github.com/modularity-modules/) space is the answer.

### Step 1: Get a set of initial modules

The Server WG requires to have at least FreeIPA, PostrgreSQL, NetworkManager, Cockpit, storaged.

### Step 2: Identify top-lvl components

This is done in the [modularity-modules](https://github.com/modularity-modules/) space.

### Step 3: Run the dependency-report

[dependency-report](https://github.com/fedora-modularity/dependency-report)

Adam is running that periodically - output is saved in the repo. Everyone can run on their own.

### Step 4: Identify new modules, extend components of existing ones...

An example: [freeipa](https://github.com/fedora-modularity/dependency-report/blob/master/modules/freeipa/x86_64/runtime-binary-packages-short.txt) - do you see something that could be a new module? Hint: Python 2, Python 3, Java, ...

### Step ?: Keep iterating until you're done! 

Keep identifying new modules, defining their top-lvl components, resolving dependencies... When it feels like we have a solid base, we can start generating modulemd files and building them.


## Other tools

[Guide for building modules](https://docs.pagure.org/modularity/development/building-modules.html)

 * [guidelines](https://fedoraproject.org/wiki/Module:Guidelines)
 * [branching](https://fedoraproject.org/wiki/Changes/ArbitraryBranching#Current_status)


### Modularity tools

[Upstream repository.](https://pagure.io/modularity/modularity-tools)

```
$ sudo dnf install -y modtools

$ mod-tools rpm2module nodejs npm
```

It will get us this modulemd:

```
data:
  api:
    rpms: [nodejs, npm]
  components:
    rpms:
      http-parser: {rationale: Build and runtime dependency.}
      nodejs: {buildorder: 10, rationale: Runtime dependency.}
  dependencies:
    buildrequires: {base-runtime: f26, shared-userspace: f26}
    requires: {base-runtime: f26, shared-userspace: f26}
  description: ''
  license:
    module: [MIT]
  summary: ''
document: modulemd
version: 1
```


### Figuring out dependencies

[depchase](https://github.com/fedora-modularity/depchase)

```
$ ./depchase -a x86_64 -c ./repos.cfg.sample resolve nodejs npm
http-parser-2.7.1-5.fc26.x86_64
compat-openssl10-1:1.0.2j-6.fc26.x86_64
pcre-8.41-1.fc27.x86_64
p11-kit-trust-0.23.7-1.fc27.x86_64
sed-4.4-1.fc26.x86_64
libicu-57.1-4.fc26.x86_64
libtasn1-4.12-1.fc27.x86_64
grep-3.1-1.fc27.x86_64
ca-certificates-2017.2.16-2.fc27.noarch
libuv-1:1.13.1-1.fc27.x86_64
filesystem-3.3-1.fc27.x86_64
libacl-2.2.52-16.fc27.x86_64
tzdata-2017b-1.fc27.noarch
libatomic_ops-7.4.6-1.fc27.x86_64
fedora-release-27-0.2.noarch
make-1:4.2.1-2.fc26.x86_64
libattr-2.4.47-19.fc27.x86_64
fedora-repos-27-0.2.noarch
fedora-repos-rawhide-27-0.2.noarch
npm-1:5.3.0-1.8.2.1.1.fc27.x86_64
coreutils-8.27-13.fc27.x86_64
...
```


## `mbs-build`

[Upstream repository](https://pagure.io/fm-orchestrator/)

Documentation how to do:
 * [Local builds](https://docs.pagure.org/modularity/development/building-modules/building-local.html)
 * [Infrastructure builds](https://docs.pagure.org/modularity/development/building-modules/building-infra.html)


## Let's create autotools module!

### Step 0, naming

What will be the name of our module? What streams will we provide?

Good name for stream (dist-git branch) is a version of the software. Let's have a look at latest versions of packages we want to put into the module:

* `autoconf-2.69-25`
* `automake-1.15.1`
* `libtool-2.4.6`

Versions don't seem to be in sync.


### Step 1, modulemd

What should be in the module? `automake`, `autoconf` and `libtool`, right?

```yaml
document: modulemd
version: 1
data:
    summary: GNU toolchain (autoconf, automake, libtool)
    description: >
        This module is composed of well-known GNU tools, such as autoconf,
        automake and libtool.
    license:
        module:
            - MIT
        content: []
    references:
        community: https://github.com/modularity-modules/autotools
        documentation: https://github.com/modularity-modules/autotools
        tracker: https://github.com/modularity-modules/autotools/issues
    components:
        rpms:
            autoconf:
                rationale: Primary component of this module.
                ref: master
            automake:
                rationale: Primary component of this module.
                ref: master
            libtool:
                rationale: Primary component of this module.
                ref: master
```

What's missing?


### Step 2, platform module

Dependencies, right!

```yaml
document: modulemd
version: 1
data:
    summary: GNU toolchain (autoconf, automake, libtool)
    description: >
        This module is composed of well-known GNU tools, such as autoconf,
        automake and libtool.
    license:
        module:
            - MIT
        content: []
    dependencies:
        buildrequires:
            platform: master
            host: master
        requires:
            platform: master
            host: master
    references:
        community: https://github.com/modularity-modules/autotools
        documentation: https://github.com/modularity-modules/autotools
        tracker: https://github.com/modularity-modules/autotools/issues
    components:
        rpms:
            autoconf:
                rationale: Primary component of this module.
                ref: master
            automake:
                rationale: Primary component of this module.
                ref: master
            libtool:
                rationale: Primary component of this module.
                ref: master
```

So let's build:

```
$ mkdir autotools
$ cd autotools
$ vim autotools.yaml
<shift-insert>
$ git init . && git add . && git commit -m init
$ mbs-build local
```

Seems like the build fails with:

```
 # /usr/bin/dnf builddep --installroot /var/lib/mock/module-autotools-master-20170824142446-Thread-25/root/ /var/lib/mock/module-autotools-master-20170824142446-Thread-25/root//builddir/build/SRPMS/libtool-2.4.6-20.module_2503b24e.src.rpm
Last metadata expiration check: 0:00:00 ago on Thu 24 Aug 2017 04:29:40 PM CEST.
No matching package to install: 'autoconf'
No matching package to install: 'automake'
No matching package to install: 'help2man'
Package libstdc++-devel-7.1.1-7.module_6faa4f4e.1.x86_64 is already installed, skipping.
Not all dependencies satisfied
Error: Some packages could not be found.
```

Where do we get those dependencies?


### Step 3, bootstrap module

```yaml
document: modulemd
version: 1
data:
    summary: GNU toolchain (autoconf, automake, libtool)
    description: >
        This module is composed of well-known GNU tools, such as autoconf,
        automake and libtool.
    license:
        module:
            - MIT
        content: []
    dependencies:
        buildrequires:
            bootstrap: master
        requires:
            platform: master
            host: master
    references:
        community: https://github.com/modularity-modules/autotools
        documentation: https://github.com/modularity-modules/autotools
        tracker: https://github.com/modularity-modules/autotools/issues
    components:
        rpms:
            autoconf:
                rationale: Primary component of this module.
                ref: master
            automake:
                rationale: Primary component of this module.
                ref: master
            libtool:
                rationale: Primary component of this module.
                ref: master
```

(running `mbs-build local` here for the first time downloads all 15000 packages of bootstrap module)

Does the build pass?

```
Finish: rpmbuild automake-1.15.1-2.module_ac4d13ab.src.rpm
Finish: build phase for automake-1.15.1-2.module_ac4d13ab.src.rpm
INFO: Done(/home/tt/modulebuild/builds/module-autotools-master-20170824143320/results/Thread-26/automake-1.15.1-2.module_ac4d13ab.src.rpm) Config(mock-Thread-26) 4 minutes 48 seconds
INFO: Results and/or logs in: /home/tt/modulebuild/builds/module-autotools-master-20170824143320/results/Thread-26
INFO: Cleaning up build root ('cleanup_on_success=True')
Start: clean chroot
DEBUG: kill orphans
DEBUG: child environment: None
DEBUG: Executing command: ['/bin/umount', '-n', '/var/lib/mock/module-autotools-master-20170824143320-Thread-26/root/var/cache/dnf/'] with env {'TERM': 'vt100', 'SHELL': '/bin/sh', 'HOME': '/builddir', 'HOSTNAME': 'mock', 'PATH': '/usr/bin:/bin:/usr/sbin:/sbin', 'LANG': 'en_US.utf8'} and shell False
DEBUG: Child return code was: 0
```

Indeed.

Can we install the module?


### Step 4, runtime dependencies

Ehm, I don't know.

So, what binary RPMs do we need?

[These.](https://github.com/fedora-modularity/dependency-report/blob/master/modules/autotools/x86_64/standalone-runtime-binary-packages-short.txt)

How do we figure out in which modules these are available? And are they actually in some module?

[This file could help.](https://github.com/fedora-modularity/dependency-report/blob/master/global_reports/all-binary-pkgs-occurrences.txt)


### Step 5, the review

As described in [the official review process](https://fedoraproject.org/wiki/Module:Review_Process), you need to [create a new bug](https://fedoraproject.org/wiki/Module:Review_Process#Module_Metadata_Files) to initiate the review process.

[It may look like this one.](https://bugzilla.redhat.com/show_bug.cgi?id=1484409)

We suggest to go to Freenode IRC channel `#fedora-modularity` and find a reviewer.


## Testing

~~Modularity testing framework~~ [Meta test family](https://github.com/fedora-modularity/meta-test-family)

[MTF Guide](http://meta-test-family.readthedocs.io/en/latest/user_guide/index.html)


## We need your feedback!

You can find F&Q and Feedback document over [here](https://docs.google.com/document/d/1SaGvw703pzKki2YVcGswuggcIqpwqqz5qPRc42xHXyU/edit). Feel free to ask your question and someone will answer it soon or later.

You can also open an issue at any upstream repository linked in the workshop.

Alternatively, you may ask your question on [webchat.freenode.net](https://webchat.freenode.net/?channels=fedora-modularity) and someone should answer it right away.
