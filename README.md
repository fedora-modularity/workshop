# Modularity workshop

This workshop should teach you how you can create your own modules.

The plan is to utilize existing guides and just link to them.

Before we start, feel free to look into [Stephen Gallagher's](https://github.com/sgallagher) blog posts on module packaging:
 * [Sausage Factory: An introduction to building modules in Fedora](https://sgallagh.wordpress.com/2017/05/30/sausage-factory-an-introduction-to-building-modules-in-fedora/)
 * [Sausage Factory: Advanced module building in Fedora](https://sgallagh.wordpress.com/2017/06/30/sausage-factory-advanced-module-building-in-fedora/)


## High-level agenda

1. [1-minute-intro to modularity](#1-minute-intro)
2. [modulemd](#modulemd)
3. [Workflow](#workflow)
4. [`mbs-build`](#mbs-build)
5. [Testing](#testing)
7. [Feedback](#we-need-your-feedback)


# The workshop

TODO:
* [ ] record steps of a basic module build
* [ ] mention that bootstrap module needs to be downloaded
* [ ] everyone can see my terminal (font and colors are acceptable)


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

Keep identifying new modules, defining their top-lvl components, resolving dependnecies... When it feels like we have a solid base, we can start generating modulemd files and building them.


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


## Testing

~~Modularity testing framework~~ [Meta test family](https://github.com/fedora-modularity/meta-test-family)

[MTF Guide](http://meta-test-family.readthedocs.io/en/latest/user_guide/index.html)


## We need your feedback!

Feel free to open an issue at this repo or at any upstream repository linked in the workshop.

In case you want to reach out to a human being, [Adam Samalik](https://github.com/asamalik) is the main coordinator, you can contact him [on his e-mail address](mailto:asamalik@redhat.com).
