---
layout: default
title: "The Engine We Lost: Digital Archaeology of Opera Presto"
description: "A deep dive into the leaked ~2012 source code of the Opera Presto browser engine."
permalink: /
lang: en
---

<nav id="top" aria-label="Top navigation">
  WEEK: <strong>I</strong>—<a href="{{ '/revival/' | relative_url }}" title="The Engine We're Bringing Back to Life">II</a>—<a href="{{ '/modern/' | relative_url }}" title="The Engine That Learns">III</a>—<a href="{{ '/css/' | relative_url }}" title="The Engine That Needs CSS">IV</a>—<a href="{{ '/es2015-gstreamer/' | relative_url }}" title="The Engine That Outpaced Chrome">V</a>—<a href="{{ '/dragonfly-shadow-dom/' | relative_url }}" title="Opera Dragonfly and Shadow DOM">VI</a> · <strong>English</strong> · <a href="{{ '/ru/' | relative_url }}" lang="ru">Русский</a>
</nav>

# The Engine We Lost: Digital Archaeology of Opera Presto

## Contents

- **[I. The Pre-Monopoly Era: What Opera Was and Why It's Gone](#i-the-pre-monopoly-era-what-opera-was-and-why-its-gone)**
- **[II. Digital Archaeology of Three Million Lines of Code](#ii-digital-archaeology-of-three-million-lines-of-code)**
  - [II.I How Was Opera Built?](#iii-how-was-opera-built)
  - [II.II How Was Opera Written?](#iiii-how-was-opera-written)
  - [II.III How Was Opera Tested?](#iiiii-how-was-opera-tested)
  - [II.IV Who Wrote Opera?](#iiiv-who-wrote-opera)
- **[III. A History of a Lie](#iii-a-history-of-a-lie)**
  - [III.I The Uncuttable UI Monolith](#iiii-the-uncuttable-ui-monolith)
  - [III.II Excuses Instead of Source Code](#iiiii-excuses-instead-of-source-code)
- **[IV. Architectural Diamonds (and a Bit of Madness)](#iv-architectural-diamonds-and-a-bit-of-madness)**
- **[V. Defrosting and Health Check](#v-defrosting-and-health-check)**
  - [V.I Moving to OpenSSL](#vi-moving-to-openssl)
  - [V.II Updating Third-Party Code](#vii-updating-third-party-code)
  - [V.III The Death of Pike and Other Adventures](#viii-the-death-of-pike-and-other-adventures)
  - [V.IV Museum of Dead Features](#viv-museum-of-dead-features)
- **[VI. What's Next?](#vi-whats-next)**

---

## Nulla. Disclaimer

This foreword appeared here only after several articles had turned into a series. I went back to the beginning and left the future reader an explanation of what to expect.

1. What follows is an account, at varying depths of technical detail, of how the Opera Presto browser and its Carakan JS engine are built. It started out as pure research; then the author felt like "reviving" the old program... and, as usual, the appetite came with the eating.
2. Much of what is written here is, legally speaking, let's say, ambiguous. The author lays no claim whatsoever to anyone else's property, and tries not to violate any licenses. Fully aware of the paradox of this undertaking, the author does not intend to discuss it any further, and instead surrenders entirely to the research.
3. **AI was used** in writing this text, and the author **is not going to hide it**. The author is no fan of AI slop either, so let it be said up front: AI was used in writing these texts only where it was genuinely necessary. Browser engines are real rocket science, and the Presto code is no exception. Working through years-deep layers of code in full detail is beyond any one person (which explains why so few, and such superficial, code write-ups have appeared over the years), and AI agents are the only tool that solves the problem.
4. 99% of the code was written by AI, 99% of the decisions were made by a human. The author has grounds to consider themselves a decent developer, but admits that the SOTA models from Anthropic and OpenAI did it faster and better.

And now, to the matter at hand...

## I. The Pre-Monopoly Era: What Opera Was and Why It's Gone

Once upon a time, the web was a different place. Not better, not worse—though there are those who would argue with me. But it was definitely freer, and there was more choice. Including the choice of which browser to use.

Much has been written about the first browser war and how IE destroyed Netscape. But the second browser war is somehow forgotten, even though its outcome is the reason we are now left with the all-devouring Chrome and its clones, Apple's Safari, and a Firefox that is diligently burying itself with a measly three percent of the market. Everything else is basically a rounding error.

In 2012, we still had IE (who could have guessed that Microsoft would give up developing it and switch to a competitor's engine!) and Opera. An absolutely independent, unique, and genuinely beloved browser developed by the Norwegian company Opera Software ASA.

Opera was a highly innovative and, at the same time, niche product. One can argue about its exact market share, though no one usually denies that on the desktop it wasn't huge. The creators were constantly thinking about how to make the program better and more convenient—without modern corporate bullshit. Things like page tabs, speed dial, pop-up blocking, searching straight from the address bar, and much more were either invented or popularized by Opera. In the post-Soviet space, Opera earned a special kind of love for its incredibly frugal handling of traffic. Caching absolutely everything, disabling unnecessary resources, rendering as data loaded, and later, Opera Mini/Turbo—you really appreciated this if you were on pay-per-minute dial-up or per-megabyte DSL.

Opera ran on almost everything. All relevant operating systems, feature phones, smartphones, game consoles, embedded devices... On many mobile phones of that era, it was the only browser capable of displaying graphics, executing scripts, and submitting forms.

And it was a fast, economical browser. The web had already started to get heavy back then; pages were appearing that were—just imagine!—hundreds of kilobytes, even megabytes in size. Opera handled this incredibly efficiently. Toward the end of its life cycle, it was battling Chrome in terms of speed, holding its own very admirably despite a staggering disparity in team resources.

It seems surprising that with all these advantages, Opera didn't conquer the world. In a right, just world, Opera and Firefox could very well hold significant market shares right now, competing with Chrome and IE, making the web better, freer, and more convenient. In the doomed world we have created, Opera couldn't withstand the competition. The company decided to stop fighting, switched to WebKit (and later to its fork, Blink), and sold its intellectual property. The name "Opera" remains, but now it's just another Chinese Chromium reskin—a decent one, they say. Former Opera employees are developing their own browser, Vivaldi—also a Chromium reskin, and also supposedly good.

But the Opera Presto source code, despite the pleas of the community, remained closed. I'll talk about why later.

In November 2016, Opera Software ASA ceased to exist as a single entity, and in January 2017, someone "leaked" the ~2012 source code into the public domain. We don't know who did it, but thank you.

## II. Digital Archaeology of Three Million Lines of Code

As I write this, it is mid-2026. 14 years is several generations of development in standards, hardware, source code, principles, and approaches. Digging into these sources is akin to excavating Pompeii; preserved under the ash of time is a cross-section of the developers' mindset from that era and the technological approaches that made the browser unique.

A few numbers to calibrate the scale. In the source tree:

- **~3.4 million lines of C++** — 2.31M in the core, 0.58M in non-core code, 0.53M in platform code
- **~9,800** `.cpp`/`.h` files
- **105 core modules**, 17 non-core components, 15 platform directories
- **1,354 test files**
- **279 Python 2 scripts** and **22 Pike scripts**

There is some documentation right in the repository. It's in an unfinished state, with links to an internal wiki, but how complete that was—we'll never know. There are absolutely no instructions on how to actually build the thing, which was the first thing I had to figure out.

Now, watch closely: **the codebase is unified across all platforms.**

The browser that ran on Windows 7, macOS, QNX, and a Sony Ericsson feature phone was built from a single repository (by the way, judging by some artifacts, they used SVN).

How was this achieved? With their own custom build system.

### II.I How Was Opera Built?

Essentially, there is no separate "Windows version" or "QNX version" in the repository—the necessary code is assembled on the fly. A build script iterates through all modules, reading their metadata (simple text files containing descriptions of which files to compile, what APIs the module exports and imports, and much more). By comparing the metadata with a given configuration, the script ultimately generates `*_jumbo.cpp` files—and these are what actually get compiled.

Modules do not reference each other by name. A module, as mentioned above, declares its API imports/exports and the conditions under which it needs them, for example:

```
API_CACHE_URL_STREAM
    URL_DataStream helper.

    Import if: FEATURE_EXTERNAL_SSL
```

Conditions are boolean expressions over features and tweaks. The build script parses them into an AST, resolves them transitively (if A imports an API from B, and that API depends on feature C, then building A implicitly requires C), and turns them into preprocessor `#define`s. The result is a declarative dependency graph: modules describe *what* they need, and the build system calculates *how* to link them. For an engine with thousands of compile-time switches, this prevents an exponential explosion of makefiles.

What are the aforementioned jumbo files? This is a local implementation of a unity build. Usually, every `.cpp` file is a separate translation unit. The compiler runs on it individually, parses all the headers it includes from scratch, and outputs a separate `.o` object file. When there are thousands of files and the headers are heavy, this results in colossal redundant work: the same large header is parsed thousands of times.

A Unity build glues many `.cpp` files into a single translation unit—literally via `#include`. The compiler runs once on the entire batch: shared headers are parsed once, the optimizer sees all the code in the batch at once (meaning it can inline across file boundaries), and the linker receives tens of object files instead of thousands. Compilation speeds up dramatically.

In Presto, it looks like this—the `operasetup.py` generator outputs `*_jumbo.cpp`, which simply includes the source files:

```cpp
// This file is automatically generated by operasetup.py
#include "core/pch_jumbo.h"
#include "modules/content_filter/content_filter.cpp"
#include "modules/content_filter/content_filter_module.cpp"
```

The price you pay for this is isolation. Inside a glued unit, file boundaries disappear: two `static void helper()` functions in different files, two symbols in anonymous namespaces, a file-scoped `#define`—everything that was local in isolation starts to conflict after gluing. Therefore, unity builds require strict naming discipline and careful macro management. Incrementality also suffers: you edit one file, and the entire jumbo unit is recompiled.

In CMake, the `UNITY_BUILD` option, which does exactly this, appeared at the end of 2019. Presto was using this technique as a standard operating mode back in the 2000s.

The second layer is the actual builder, **Flower**, living in `platforms/flower`. It's a build system in Python 2 designed to replace `make`. Its documentation explicitly criticizes `make` on four points:

- `make` builds the dependency graph in advance, whereas in Presto, the list of source files is unknown until the setup phase;
- Configuration in `make` consists of flat strings with no lists or dictionaries;
- The output of a parallel `make` build is unreadable;
- `make` targets are not self-documenting.

The name of this build system was likely chosen based on its architecture. Flower is built on Python generators. A node is a unit of work, and a **flow** is a generator function describing how to build that node. `yield` pauses the flow until dependencies are ready—achieving cooperative multitasking without threads.

In MSBuild, the logic is similar—the entry point is the `operasetup` project, which executes the exact same hardcore scripts. It has its own quirks, but overall, it works.

But that's not all. Presto is configured **entirely at compile time**. No runtime flags, no feature flags in the modern sense. There are four mechanisms:

1. **Features** — large functional blocks toggled on or off as a whole. They are described in `modules/hardcore/features/features.txt`—there are **416 features**. Each entry contains a name, the responsible engineer, a description, the resulting `#define`s, and dependencies. This file is used to generate profiles—`profile_desktop.h`, `profile_minimal.h`, etc.—and the minimal embedded profile disables about 70% of what the desktop profile enables.<br />
Each feature in the registry is a little questionnaire: description, owner, what it defines, what it depends on, and an "on/off" matrix across all five profiles. Reading `features.txt` is like flipping through a ship's log: here is `FEATURE_OUT_OF_MEMORY_POLLING` — "enable if you have a lot of memory, but it's still limited"; here is `FEATURE_LIMITED_FOOTPRINT` — "reduces code size at the cost of slower algorithms."

2. **Tweaks** — constants (`TWEAK_*`), of which there are about **1,200** across the tree. Each tweak can have a different value for different profiles:

   ```
   TWEAK_HC_FREE_MESSAGE_POOL_INITIAL_SIZE   jl
       Value                   : 64
       Value for desktop       : 512
       Value for minimal       : 16
   ```

   This is essentially performance tuning baked into the binary at build time.

3. **Capabilities** — macros like `*_CAP_*` (about **1,800** of them) that solve the problem of compatibility between module versions. Instead of "if module version ≥ X," you write "if `DPI_CAP_PLATFORM_EXECUTE` is defined." A module declares what it can do, and the consumer checks it via the preprocessor.

4. On top of all this lie product overrides in `platforms/*/product/` — separate `features.h`, `tweaks.h`, `system.h`, `config.h` files for each platform.

The philosophy is clear: configure everything at compile time so that disabled code is simply *absent* from the binary, rather than checked at runtime. Zero cost for disabled features. For embedded devices in the 2000s, this was absolutely the right decision.

The downside is also clear. First, the combinatorial explosion: 416 features multiplied by profiles yields an unfathomable configuration matrix, and a bug in one configuration might not appear in another. Second, **code vanishes en masse behind macros**. The `FEATURE_*` names themselves rarely appear in the source code—during build generation, a feature expands into its derivative defines (usually like `*_SUPPORT`), and the entire engine is paved with them: there are nearly fourteen thousand `#ifdef *_SUPPORT` checks in the tree, and almost seventy thousand preprocessor conditions in total. You change something, and it "doesn't work" because the entire block is stripped out by the preprocessor in that specific configuration, and you only figure that out during testing. Judging by leftover code artifacts, developers ran into this:

```cpp
// Left-over from FEATURE_SCRIPTS_IN_XML, to be removed when no longer used.
#define SCRIPTS_IN_XML_HACK
```

The feature was killed off, but the hack it once enabled lived on.

### II.II How Was Opera Written?

The layers of the browser's code can be studied endlessly. We can't view the commit history, we don't have the archive of the internal issue tracker, and we have no access to internal documentation, if it even existed. But the development culture typical of that era left its artifacts: issue tracking right in the comments, unfinished snippets of documentation next to the code, various explanations, copyrights—despite the incredibly high engineering standards, this was par for the course back then. Thanks to this, a lot becomes clear.

The oldest code dates back to 1995: 2,733 files bear a copyright with this date—this is the "Big Bang" of the engine. Technically, the oldest line of code in the tree is from 1984: a GNU Bison skeleton hardcoded into the generated CSS parser. The oldest **living** code, however, is from 1990, and it has a great backstory, which we'll get to shortly.

Some of this documentation (literally starting with "not much here yet") explains the development principles Opera adhered to for many years. Let me quote two lines that are the quintessence of these principles:

> **It sounds silly even to mention it, but we do require C++ to be supported. Not every interesting platform has C++.**

and

> **Some advanced C++ concepts are not required.**<br />
> **RTTI, STL, namespaces, exceptions: we don't need them.**

Again, remember where Presto ran. The engine's feature registry lists five build profiles: desktop, smartphone, tv, mini, minimal. The exact same code was compiled into a desktop browser, a browser for feature phones, firmware for TVs and set-top boxes (there are still 71 files in the tree mentioning ATVEF—the interactive television standard), and the server-side renderer for Opera Mini. Looking at the code fossils, you can reconstruct the whole zoo: Symbian/EPOC (34 files), Qualcomm BREW (60), QNX and its Photon GUI, Windows CE. The compilers for these platforms were fitting: who needs the STL when it's 2002 and your C++ is just "C with classes" built by a vendor's phone toolchain.<br />
The phrase "interesting platform" is a hyperlink. It leads to the website for Plan 9 from Bell Labs. Opera's engineers seriously kept in mind that somewhere out there was an interesting platform that didn't even have C++.

The "we don't need the STL" decision had a consequence that defined the entire look of the code: **Opera loved to write everything themselves** (and even the things they didn't write, they often heavily modified). There is an `stdlib` module in the tree—their own implementation of the C standard library: `op_strlen`, `op_snprintf`, `op_memcpy`, and so on. They have their own allocator and memory manager (`modules/memory`), their own strings (`OpString` on top of their custom `uni_char` type—UTF-16, long before `char16_t` appeared in the standard), their own containers (`OpVector`, `OpHashTable`), and their own smart pointers (`OpAutoPtr`).

In fact, they wrote their own containers **twice**. In the `otl` module—the Opera Template Library—lies the second generation, and its documentation explains why: the old `OpVector<Foo>` stored `Foo*` pointers; the new `OtlVector<Foo>` stores values—"STL-like containers for core." So, a culture that proclaimed "we don't need the STL," over the years, came to the point where they wrote their own STL, then realized they wrote it wrong, and wrote another one. Nearby is a third pass at a related topic: the `opdata` module with copy-on-write buffers. Reinvented wheels of different generations, all parked in the same garage: OTL took root sporadically in newer subsystems, OpData in `scope`/protobuf and network buffers, while ~90% of the code still lives on `OpVector`/`OpString`.

And now for the promised 1990 code. Formatted output—`op_sprintf` and friends. The file header preserves its pedigree: "This code is taken from the EMX run-time library… Copyright (c) 1990-1997 by Eberhard Mattes," with an addendum: "Converted to joint ascii/unicode printf by lth@opera.com / November 2005." Code written for the OS/2 GCC port runtime in 1990 was taught Unicode in 2005, and it still prints every single line in this browser today.

The random number generator is also a bit of a patchwork—a Mersenne Twister, Copyright 1997–2002 Matsumoto and Nishimura. All of this is meticulously documented in the feature registry—to the point where trying to compile the engine without the required flag hits a wall of `#error`s recounting the history of the code's origin.

But this isn't the only length Opera went to for the sake of universality. This code has its own OOM (Out-Of-Memory) discipline, exotic even for its time. The engine was written for devices where 16 megabytes was a luxury, and **every single** allocation in it is checked. No exceptions—instead, they have their own `LEAVE`/`TRAP` mechanism: `setjmp`/`longjmp` with a manual cleanup stack. The documentation mentions this as a traumatic experience:

> A typical example of something that was outside the envelope of change is the out-of-memory handling introduced in Opera 6 — it affected virtually every line of core code and was quite costly to implement.

"Affected virtually every line of core code." That's not a figure of speech: there are about 9,700 places in the tree using `LEAVE`/`TRAP`, and almost every function returns an `OP_STATUS` that cannot be silently ignored.

The mechanism itself lives in `modules/util/excepts.h` (the architect is listed as Petter Reinholdtsen), and its origin can be read in the names: the `LEAVE` macro internally calls `User::Leave(i)`—which is verbatim the EPOC OS API. The Norwegians modeled their error handling after a phone OS: `TRAP` is `op_setjmp` on top of a chain of `CleanupItem`s, and classes like `ANCHOR` register objects for automatic cleanup when "jumping out." Essentially, they are manually implemented exceptions, without compiler support, but guaranteed to work on any 1999 toolchain.

And this wasn't a cargo cult: the memory module can run the engine on an artificially constrained heap (via `dlmalloc`) and deterministically drive it to OOM on a desktop—"a powerful combination allowing accurate testing of OOM situations," as the docs say. So, this was actually tested.

Interestingly, there was an attempt to transition to native C++ exceptions: the code includes defines for this logic that were never used. I assume that by 2012, supporting such incredibly weak devices lost its purpose, and newer compilers no longer added binary bloat due to exceptions. The developers then tried to use idiomatic language features, but never finished the job, because rewriting three million lines of code riddled with OOM checks is a task requiring hundreds of man-months.

By the way, judging by the modelines, they wrote this in emacs/vim. Pure console, pure hardcore!

### II.III How Was Opera Tested?

After everything you've learned, you won't be surprised: Opera's testing system was also entirely homegrown. Tests are written in `.ot` ("Opera Test") files inside the modules. It's a domain-specific language supporting `setup` blocks, data tables, and iterations. Tests can be written in C++, ECMAScript, and can even include inline HTML blocks.

A Pike script, `modules/selftest/parser/parse_tests.pike`, compiles `.ot` into C++, which is linked **directly into the browser itself** when the `FEATURE_SELFTEST` feature is enabled. Tests are not run as a separate binary; they are executed by a dispatcher inside the running browser—on desktop, this looks like Opera launching and opening some windows inside itself.

How many tests are there? **1,354 `.ot` files**. The coverage is uneven but logical—the rendering core is covered best:

- `layout` — 55 tests
- `svg` — 52
- `dom` (core + html) — approx. 80
- `style` — 32
- `logdoc` — 29
- `url` — 28
- `forms`, `util`, `cache`, `pi` — 23–26 each

Left without tests were `externalssl`, `webgl`, `applicationcache`, `obml_comm`, `xmlparser`, `updaters`, plus third-party libraries. In other words, the team tested critical subsystems seriously, while leaving the periphery and borrowed code alone.

For its time, this was a respectable testing culture, but by today's standards, it has fundamental limitations. The tests are essentially integration tests—they run inside the assembled browser, with no unit-test level isolation. To build them, you need a working interpreter for **Pike**—a language even more exotic than Python 2. And there was no CI in the modern sense: tests were run locally inside the build, not on every cloud commit.

### II.IV Who Wrote Opera?

As a good historian once said: "We cannot get inside these people's heads." But by the traces they left behind, we can picture them very clearly. Take the internal code names, for example:

The engine itself is **Presto**, "quickly," like the musical tempo. Opera's JavaScript engines were a dynasty named after ancient scripts: **Linear A** and **Linear B** (Aegean scripts), then **Futhark** (runes), then **Carakan** (Javanese script).

Browser releases in the 9.x-10.x era were named after falcons (Merlin, Kestrel, Peregrine), and later came the fish (Barracuda, Swordfish, Wahoo — 11.x–12.00).

The graphics library is called VEGA, an acronym: "Library for **VE**ctor **G**raphics with **A**nti-aliasing." The JPEG decoder is called `jaypeg`, the PNG decoder is `minpng` ("focusing on minimal footprint"), and the password manager is `wand`. The widgets module is called `gadgets`, and its `module.about` explains why: "The module is called gadgets because, in the code, 'widgets' already has another meaning." And the strange name `logdoc` for the core DOM tree is simply "logical document."

A special mention goes to an Easter egg in the CSS parser. In the grammar, among other tokens, there is `CSS_TOK_DOUBLE_RAINBOW_FUNC`: Opera 11.60 introduced the `-o-double-rainbow()` gradient function. A Double Rainbow, hardcoded into the Bison grammar to allow developers "to sprinkle some vivid color into their web projects." It was elegant mockery of the standards and proof that the browser was made by real people with a great sense of humor, who clearly enjoyed their work.

But the most "human" layer of this archaeological dig lies in the comments left in the code. 1,409 TODOs, 1,859 FIXMEs, 1,449 XXXs, ~850 links to the internal `CORE-xxxxx` bug tracker, and ~690 to the desktop `DSK-xxxxx` tracker. But statistics don't convey tone. Here are my favorite finds:

```cpp
// Warning: some cargo cult programming ahead.

if(!this)   // I know, it is silly...  :-( but if it prevent the crash, it's ok...

// Quirk crap. Try not to look.

// horrible hack, but it's probably not a good idea to font switch to ahem. ever.

// Rebuild the widget from the beginning. Yes, this sucks.

if ((p = strstr(head.longname, "?B?")) != NULL) // It's NOT Base64, it's QP damn it!

// Now we're screwed, there is no way out. Let's hope the error status...

const LayoutCoord o(0); // Avoid an insane amount of "LayoutCoord(0)" below.

// Sorry, this is ugly, but otherwise we crash

OP_ASSERT(!"What to do, what to do, with this strange content type");
```

It would be unfair to reduce this archaeology strictly to curiosities. Inside Presto lie things that still command professional respect today.

**Carakan** — the JavaScript engine — is a register-based virtual machine with JIT compilation to native code for **four architectures**: x86, x86-64, ARM (including Thumb), and MIPS; nearby in the utilities are assemblers for PowerPC and SuperH. Values are NaN-boxed (8 bytes on 32-bit platforms), and objects live on "hidden classes"—internal documentation calls this the Compact Object Model and boasts about reducing the overhead from 6–7 pointers per object down to one. The JIT performs type profiling directly in the bytecode and recompiles hot code as the picture becomes clearer. The regular expressions are also homegrown, and they have **their own separate JIT** for those same architectures. World-class work, executed by a single small team on their own infrastructure, without LLVM, in 2009.

**VEGA** — a vector rasterizer with a software backend and hardware ones: Direct3D, OpenGL, DirectFB. On top of it sit Canvas 2D, WebGL (with a fully custom parser and GLSL translator in `modules/webgl`—they wrote their own shader compiler), SVG with SMIL animation, and filters capable of rendering on the GPU.

**HTML5 Parser** — a complete implementation of the spec algorithm of that time: tokenizer, tree builder with all insertion modes, foster parenting, adoption agency, and a speculative pre-parser for resource prefetching. Internal name: Ragnarok. Presto wasn't just fast; it was a highly modern engine.

**Text.** A custom OpenType shaper with support for Arabic and Indic scripts (without HarfBuzz—it barely existed then), a custom implementation of the Unicode Bidirectional Algorithm, Unicode 6.1 tables, a charset detector, and three dozen charset decoders. All their own.

**Network.** Again, all homegrown: HTTP/1.1 with aggressive pipelining (the code houses flags like `tested_http_1_1_pipelinablity` and a `pipeline_problem_count` counter: the engine tested every server for compatibility and backed off if issues arose), SPDY versions 2 and 3, WebSockets complying with the final RFC 6455, and CORS with beautifully implemented preflight. And FTP. And a Gopher stub—obviously.

And the **CSS pioneering** for which web developers loved (and sometimes hated) Opera: `-o-tab-size`, `-o-object-fit` (`object-fit` was invented at Opera), `@viewport` (a CSS alternative to the viewport meta tag), and GCPM floats for print layouts. In the properties table, two generations of flexbox peacefully coexist within a single file: the final `display: flex` from 2012 and the ancient `-webkit-box` from 2009.

If you stack all these layers together, a portrait of the team emerges from the excavation. People who seriously intended to run on every platform in the Universe—and therefore wrote everything for themselves: libc, STL, an allocator, a font shaper, a shader compiler, four JIT backends. People with almost military discipline regarding metadata—who also left lively, funny comments. People who rehearsed out-of-memory scenarios on flip phones—and hid a double rainbow in the CSS grammar. This was a vast, intelligent, distinctive engineering world, populated by people who loved what they did.

## III. A History of a Lie

After Opera ASA decided to switch to WebKit, a lie was born, rivaling the infamous "640K ought to be enough for anybody." Users—whose core demographic was people with an IT background—were told (and I quote): ["all these changes will happen under the hood for regular users"](https://habr.com/ru/companies/opera/articles/169239/).

People took this to mean "instead of Presto there will be WebKit, instead of Carakan there will be V8, but all the features I chose this browser for will remain." And there were a lot of such features, some of which have still never been implemented anywhere else.

Thirteen-year-old spoiler alert: we were tricked. Bam-boo-zled. Played like a fiddle.

Soon enough, the phrase "changes under the hood" became a sad meme in the community, and Opera became just another Chrome reskin. Well, you know this story; I won't dwell on it now.

### III.I The Uncuttable UI Monolith

Could they have actually done it exactly as initially announced? No, and the reason stems from that exact same principle of "keeping everything in-house." Opera's desktop interface was called **Quick**, and it was built using an entirely custom-drawn set of widgets. Opera did not use the operating system's buttons, lists, or input fields—it drew them all itself. `OpButton`, `OpTreeView`, `OpEdit`, `OpToolbar`, `OpWorkspace`, `DesktopWindow`, `BrowserDesktopWindow` — these were all rendered by the widget engine (`modules/widgets`) on top of the same `VisualDevice` used for web pages, while the visual appearance was pulled from the skinning system (`modules/skin` and `OpSkinManager`).

This is exactly why Opera 12 looked identical on Windows, Mac, and Linux, and was fantastically customizable—the famous Opera skins altered the entire interior beyond recognition. Every pixel of the interface was drawn by Opera itself, requiring very little from the OS. The platform layer (`pi` plus `platforms/unix`, `platforms/windows`, `platforms/mac`) only provided:

- a top-level native window
- a message loop
- system dialogs (file picker)
- clipboard access
- drag-and-drop
- native context menus

Everything else—tabs, panels, the address bar, sidebars, settings—was drawn by Quick. Roughly speaking, Opera's interface was about 90% platform-independent, with `platforms/` serving as a thin shim.

The code does show an intention to create an interface between the engine and the UI: the `OpWindowCommander` class provides a clean, narrow embedding API. But this boundary was routinely bypassed, and Quick modules dug straight into the engine's guts, using its internal classes and objects. Ultimately, `WindowCommander` became less of an embedding API and more of an event notification layer.

But that's not all. The UI and the engine shared **absolutely everything**: the same build system, a single shared binary, the exact same error-handling idioms (`OpStatus`, `LEAVE`/`TRAP`), the same `OpString`, a single global `g_opera`, and the same `pi` layer. Quick wasn't an application embedding a web engine via a stable interface. Quick was **compiled directly into the engine**. It was a single monolithic executable where the UI code called the engine directly, like a neighboring module.

Therefore, technically speaking, "keeping the interface and swapping the engine" meant "rewriting the entire interface from scratch": ripping out all direct calls to `Window`, `FramesDocument`, and `VisualDevice`, replacing the document model, rewriting the bindings for a foreign engine, and completely overhauling the build system. You absolutely cannot pull off a change like that "under the hood."

### III.II Excuses Instead of Source Code

But why not just open-source the code that nobody needed anymore? Who would that have hurt?

Vadim Makeev, Opera's evangelist at the time (and, by the way, the author of that infamous "under the hood" blunder), cited three reasons:

1. **Poorly documented code** (which would have to be written from scratch for the open-source community).<br />
This is a relative argument—there is documentation, albeit incomplete and sometimes outdated. Would this have been a real blocker for the community back then? I won't judge, but the code itself is highly self-descriptive and cohesive; it follows a hexagonal architecture in spirit and approach. Plus, it has excellent test coverage. Digging into it is genuinely interesting.
2. **Lack of financial benefit** (preparing the release would require a year of work by a full team, a pointless waste of money for a commercial company).<br />
No direct lie here either. On the other hand, dumping "unprepared" source code wouldn't exactly have caused financial ruin, either.
3. **Licensing restrictions** (the Presto core was heavily tied to closed, proprietary third-party libraries that would need to be painstakingly excised).<br />
This was the strongest argument: the repository mentions **45 third-party components** under a tangled web of incompatible conditions. Some just require attribution (OpenSSL, Unicode data, Xiph codecs). Some are copyleft, which "infects" distribution (a variant of GPLv2 for FreeType, a triple GPL/LGPL/MPL license for Hunspell, LGPL for GStreamer). And some were **commercial, proprietary foreign code that Opera had absolutely no legal right to publish**: the Monotype iType font engine (`FEATURE_3P_ITYPE_ENGINE`) and the Matrix SSL library (`FEATURE_3P_MATRIX_SSL`). On top of that lay non-distributable data: a root certificate store with trusted anchors, search engine data, and ~197 MB of translation databases tied to Opera's internal infrastructure.<br /><br />
But this argument loses its teeth once we look at the leaked code. It only contains the desktop versions for Windows, Linux, and macOS.<br />
**Monotype iType** was used on devices lacking FreeType and system fonts. **Matrix SSL** was a compact commercial TLS for embedded devices. That's it; the rest of the components are under free licenses. The only definitively non-distributable things left were the root certificates, which are trivially easy to strip out, just like the other non-public data. This absolutely does not look like something requiring months of work from lawyers and engineers.

**Opera ASA could have open-sourced the desktop versions of the browser at any moment, but they chose not to.**

## IV. Architectural Diamonds (and a Bit of Madness)

As you have gathered, Opera preferred to do everything themselves whenever possible. When that wasn't an option, third-party code was integrated with no external dependencies, and sometimes with their own modifications. There's quite a lot of this code: `sqlite`, `zlib`, `libfreetype`, etc. The largest and most interesting borrowing is a fork of OpenSSL, which they heavily modified. The module is called **libopeay**, which breaks down as **lib + op**(era) **+ eay** — the initials of Eric A. Young, the author of **SSLeay**, the library that OpenSSL evolved from in 1998. The module's artifacts show that the code was ported over multiple times, starting from the late SSLeay era (versions 0.8.x) up to the then-current OpenSSL 1.0.0g. They didn't use the upstream source wholesale: a dedicated script carved out everything unnecessary—apps, demos, test, documentation, platform wrappers for VMS and OS/2. Then another script wrapped **each** remaining `.c` file into its own `opera_*.cpp` file—otherwise, thousands of OpenSSL C files simply wouldn't have survived the jumbo build process with its single translation units and name conflicts. After that, it was clearly massaged heavily by hand. So, even "taking something off the shelf" at Opera meant running someone else's code through their own meat grinder.

But out of all of OpenSSL, Opera only used the cryptography: bignum math, RSA, DH, ciphers, hashes, X.509, and ASN.1. **The TLS protocol itself—handshakes, records, sessions—was written from scratch, entirely in-house.** OpenSSL ships with its own SSL/TLS implementation (the `ssl/` directory, files like `s3_clnt.c`, `t1_enc.c`), and they *are* in the tree, wrapped in `opera_s3_clnt.cpp`... but they are marked with the `#[external-ssl-only]` tag. On desktop, they **were never compiled at all**. The desktop browser encrypted traffic using its own custom stack—the **libssl** module.

Why write your own TLS? For the same reason you write your own STL and libc: control and portability. A custom implementation adhered to Opera's OOM discipline, its asynchronous model, and its certificate manager and dialogs. The boundary is declarative: libssl asks libopeay for a crypto primitive API, and nothing more. Furthermore, libssl has extensions that OpenSSL lacked, or that Opera bolted on in its own way:

- **Asynchronous RSA key generation.** The engine is single-threaded, and generating a 2048-bit key takes a long time. Therefore, `asynch_rsa_gen` is not a function, but a **state machine**: `RSA_KEYGEN_INITIALISING → GENERATING_P → DONE_GENERATING_P → ...`. Key generation is split into steps, between which the engine processes messages.
- **EV certificates.** The `cert_ex_data` file (2007–2008) attaches Opera's Extended Validation data to X.509 structures via the `CRYPTO_EX_DATA` mechanism. Opera extended someone else's structure without touching its code.
- **ElGamal** (`elgamal`) — which had been ripped out of OpenSSL by that time, but Opera needed it, so they brought it back.

---

The engine natively (without the Google Protocol Buffers runtime) implements protobuf. This powers the remote debugging logic, back when you could debug a page on your phone directly from your desktop. The debugger itself—Opera Dragonfly—was architecturally decoupled from the engine and ran on top of it as a web application; new versions of the debugger could be fetched from Opera's servers without updating the browser itself. The source code for this tool, by the way, [was open-sourced](https://github.com/operasoftware/dragonfly).

---

Vector graphics are handled in `modules/libvega`, and there are three backends for it: a software rasterizer (`vegabackend_sw`), 2D hardware acceleration (`vegabackend_hw2d`), and a full 3D pipeline on the GPU (`vegabackend_hw3d`)—the latter being necessary for CSS transforms and effects. Paths (`VEGAPath`) are rasterized using a sweep-line algorithm; gradients, patterns, and blend modes are supported.

Beneath that lies `modules/libgogi`—a low-level graphics layer called the "Multiplatform Desktop Environment" (MDE). This is a region-based invalidation system: it tracks screen rectangles that require redrawing and merges overlapping ones. A classic solution from the pre-GPU era, when the main goal was to minimize pushing pixels to the framebuffer.

---

The engine's core is single-threaded. There is one message loop, and everything runs through it: HTML parsing, the CSS cascade, layout, JavaScript execution, rendering. The DOM tree and the box tree are not protected by mutexes or atomics—and they don't need to be, because they are always accessed by exactly one thread. `THIS IS NOT THREAD-SAFE` comments serve as reminders of this in multiple places.

A curious detail: the platform layer has a `PI_CAP_SYNCOBJECT_REMOVED` macro—it seems Presto once had thread synchronization objects, and they **removed** them.

However, Presto does have a framework for isolating processes communicating via messages, implementing not multithreading, but multiprocessing, much like Chrome. Or rather, it *would have* implemented it; the work was never fully completed. But the approach itself is impressive.

The multiprocessing component module provides entities like `OpComponent`/`OpComponentManager`/`OpComponentPlatform`/`OpTypedMessage`/`OpChannel`. The model assumes one `OpComponentManager` per process; components communicate exclusively via `OpTypedMessage` over channels; an address is a triplet: "manager.component.channel". The communication format is **serialized protobuf** (`component.proto`: `source`/`destination Address{componentManager, component, channel}`, type, `bytes data`). The transports are exactly what you'd expect: `posix_ipc` on `*nix`, and shared memory + events on Windows.

The framework was **intended** to be broad ("thread **or** process," multiple worker types), but in the actual 12.15 codebase, the threading branch consists of stubs.

The only thing they managed to partially use this mechanism for was isolating **NPAPI** plugins (Flash Player, Java Applets, etc.): if a plugin crashed, it wouldn't drag down the engine monolith. When x64 support was added, this same mechanism came in handy for running 32-bit plugins through a wrapper.

---

Pike—a programming language you've never heard of—was widely used in Opera. Today—and even back in 2012—its use looks strange; Python seems like a far preferable choice. Why keep Pike around just to build tests when you already have Python building the code? But Opera's engineers never did anything without a reason, so I got curious and dug into it.

The first clue was in the copyrights: parts of the test compiler bear Copyright (C) 2002. This is the Opera 6/7 era. The second clue is in the feature registry: `FEATURE_3P_PIKE_UNIVERSE` — "Pike... **used in the Opera Mini servers**." The third clue: various scripts—internationalization table generators, Doxygen runners, some formatters. Pike was at Opera before Python, and it was a tool the team was accustomed to.

Why this language, and not, say, Perl? The `.ot` test format isn't just data: embedded inside the tests are blocks of **C++ and JavaScript**. To compile an `.ot` file into C++, the parser needs to know how to tokenize C-family code. And here Pike had an advantage: **its standard library already included a ready-made C/Pike tokenizer**—the `Parser.Pike` module. It provided parsing for curly braces, strings, comments, and the C++ preprocessor right out of the box.

From there, the golden rule of "if it ain't broke, don't fix it" applied. It was easier to write Flower in Python, but test compilation remained on this exotic interpreter.

Perl is still used, though; the localization build runs on it.

---

Two absolutely legendary Presto features that have never been properly replicated anywhere else are mouse gestures and fully customizable keyboard controls. From an engineering standpoint, they are beautifully implemented: both mouse gestures and keyboard shortcuts live in a unified namespace of virtual keys, `OpKey::Code`, described in `modules/hardcore/module.keys`:

```
OP_KEY_GESTURE_UP    GestureUp   gesture   (module.keys:247, + Down/Left/Right and diagonals)
OP_KEY_FLIP_BACK     FlipBack    flip      (module.keys:258, + FlipForward)
```

The gesture recognizer (`MouseGesture::CalculateMouseGesture`) turns a stroke into a direction and returns a specific `OP_KEY_GESTURE_*` code—and then feeds it into **the exact same** dispatcher as the keyboard: `InvokeKeyPressed(gesture, ...)`. Rocker gestures (holding the left mouse button and clicking the right) synthesize `OP_KEY_FLIP_BACK`/`FORWARD`. So, to the engine, a "down-left gesture" and "Ctrl+Z" are events of the exact same nature, differing only in the `ActionMethod { METHOD_MOUSE, METHOD_KEYBOARD, ... }` field.

Every module defines a list of its actions in `module.actions`, and simple `.ini` configs describe the bindings of methods to actions depending on the context:

```ini
GestureLeft, GestureDown     = Rewind, 0
GestureRight, GestureUp      = Fast forward, 0
Platform Mac, SwipeLeft      = Back
...
z ctrl                       = Undo
1 ctrl                       = Go to speed dial, 1
Feature ExtendedShortcuts, 1 = Switch to previous page
```

Users could remap absolutely everything by editing these `.ini` files.

This makes it crystal clear why these features "didn't make the cut" during the migration: this is **an entire core input subsystem**—its own virtual key namespace, a recognizer built as an extension of the keyboard, a compile-time action table, and text configuration layered on top. There is nothing like this in the Chromium model; replicating "a gesture is a key, and bindings are editable text inheriting by context" would have meant dragging in an entirely foreign input architecture. It was easier not to bother—and Opera on Blink launched without gestures, only to hack them in (partially, via extensions) much later and in a completely different way.

---

And what did all this meticulously planned Presto architecture yield? First and foremost—portability.

Let's look back at the internal documentation:

> ...there is a summary of the estimates time it takes to port Opera to a new platform, based on the ports to Kyocera and PowerTV. The figures are probably guesstimates as much as estimates, but adding them up it comes out to between 110 and 170 man-days for a port of the full browser, not including customer extras.

An engineering team could port the browser to a new platform in about a month. Not bad.

---

And finally, a word on how Opera survived financially: affiliate links. They are everywhere: hardcoded bookmarks, Speed Dial links, default search engines. Every click brought in a penny, and pennies make dollars.

This revenue stream had to be protected—not from the user, but from some rogue extension that might hijack the links. To do this, they built a mechanism called **Search Protection**: a system that *defended these settings against modification*. Search engine files were RSA-signed, the chosen default search engine was guarded by a checksum, and in the event of a mismatch, the browser silently rolled back the search setting to the signed copy.

There's plenty more fascinating and crazy stuff in the code that I want to talk about. There's enough material for three more chapters like this, but I restrained myself.

## V. Defrosting and Health Check

Building the code on Linux turned out to be surprisingly easy despite the outdated toolchain. In fact, the only exotic requirements are Python 2.7 for Flower and Pike for the tests (not counting Perl; any current version in any current distro works fine), which is easiest to just shove into a Docker image once and be done with it. I managed to build the engine itself on a modern GCC 14 after one minor fix and a dozen compiler flags:

```
-fpermissive, -Wno-narrowing, -Wno-deprecated, -Wno-register, -Wno-class-memaccess,
-Wno-deprecated-declarations, -Wno-misleading-indentation, -Wno-address,
-Wno-stringop-overflow, -Wno-stringop-truncation, -Wno-array-bounds, -Wno-restrict,
-Wno-format-overflow, -Wno-format-truncation, -Wno-nonnull, -Wno-dangling-pointer,
-Wno-write-strings
```

Building in VS 2026 required more fixes, both in the language (declarations that modern MSVC finds "incomprehensible" and other minor things) and in the tooling. For example, `libvpx` was built via integration with the `yasm` assembler. I couldn't get `vsyasm` to run, but writing a Python wrapper took ten minutes. I also had to add paths to 7zip (needed for packing skins) and the aforementioned Perl.

And that was enough. The builds compiled and ran, the tests executed and passed—except for a few that were apparently tied to internal mock servers. Almost boringly so.

It actually builds fast: on an Intel Core Ultra 9 275HX, a full single-threaded build takes just 162 seconds, hitting a wall at ~103 seconds when heavily parallelized (up to 24 threads). This modest 1.5x scaling perfectly illustrates the nature of jumbo builds, where the bottleneck shifts from CPU cores to memory bandwidth.

Did I find bugs? Of course. Let me tell you about the most interesting one: the Windows x64 build crashed on every single page, while the Linux x64 browser worked flawlessly. Digging in led to the JIT: **the trampoline transitioning from bytecode to native code didn't account for the Win64 calling convention**—which uses different registers for argument passing and requires a mandatory 32-byte shadow space that the Unix convention knows nothing about. Add to that a scattering of classic 64-bit porting blunders: `long` where a pointer is needed, truncating `HANDLE` to `int`, and a special gem—modern MSVC *optimized away to nothing* the table of JIT instruction handlers because, technically, nobody was "referencing" it.

Interestingly, starting with version 12, Opera did release x64 builds, including for Windows, but naturally, those official builds didn't have this bug.

### V.I Moving to OpenSSL

Fourteen years for a browser is several shifting technological eras. Twenty, even fifteen years ago, a new browser version could be released every few months, maybe half a year. But right around the early 2010s, the process accelerated exponentially. The web Opera was meant for simply doesn't exist anymore.

The first thing we hit is security issues. Opera 12.15 optionally supports TLS up to version 1.2, and while that is technically still enough to establish a secure connection with most websites, Opera can no longer verify a single certificate and will throw security warnings at every step.

As a temporary fix, you can disable this in the code, forcing Opera to trust all sites. As a permanent fix, you need to update the root certificates and implement TLS 1.3.

Opera's SSL stack, as you recall, is proprietary, albeit based on OpenSSL. Its ceiling is TLS 1.2 (the code for which is marked `TLS 1.2 (Jan 21, 2008: Not yet debugged)`) with RSA/DHE key exchange and CBC/RC4/3DES ciphers. Bolting TLS 1.3 with ECDHE + AEAD encryption onto this is technically possible, but it means manually recreating ten years of cryptography evolution, while an open and battle-tested OpenSSL 3 is sitting right there. Handing the TLS protocol layer off to the library looks like a quick win. Moreover, the infrastructure for this already exists—the aforementioned Matrix SSL was used in exactly this scenario.

In Presto's architecture, the TLS layer is `ProtocolComm`, a link in the connection processing chain; next to the native `SSLConnection`, I slotted in a new `OpenSSL_Record_Layer`, inside which OpenSSL operates via a memory-BIO: the engine never hands the socket to the library; instead, it pumps bytes between the library's buffers and its own asynchronous transport. All consumers up and down the stack never noticed the switch. The implantation was practically painless, except that I had to re-teach Opera how to read certificate and encryption info.

Certificate verification, however, remained on Opera's side: its store, its trust policy, its dialogs. OpenSSL handles the handshake, but the "trust or not" decision is made by the engine—using an updated set of Mozilla root certificates plus the OS system store (a minor concurrent tweak). This preserved the entire native security UI, the certificate manager, and the user experience.

It wasn't entirely without hiccups, though. Visiting a site with a self-signed certificate would "freeze" the browser. TLS wasn't to blame: the warning dialog **was appearing invisibly**—the UI was rendering it as an overlay on top of a page that didn't exist yet (since the connection was paused during the handshake). A classic UI bug: "the window is there, but nobody sees it," which existed dormant in Quick UI because the native stack simply never reached that dialog under those specific conditions.

And then came the best part. Once the protocol migrated to OpenSSL 3, **the entire frozen copy of OpenSSL 1.0.0g was no longer needed**. The commit deleting it was **2,119 files and nearly 470,000 deleted lines of code**. Out of libopeay, 127 files remained: headers and Opera's addon extensions—but even those weren't thrown away; they were **ported to OpenSSL 3**: asynchronous key generation is now `asynch_rsa_gen_ossl3.cpp` sitting on top of the EVP API, and EV data and purposes moved to the new X.509. The stuff Opera bolted on top of someone else's library was carefully preserved.<br />
As a final touch, a build switch was added, and now OpenSSL can be compiled statically or loaded from system libraries.

<p style="text-align: center;">
  <img src="{{ '/img/tls1.3.png' | relative_url }}" alt="TLS 1.3 and new certs logic are working" />
</p>

### V.II Updating Third-Party Code

Aside from the massive OpenSSL fork, Presto is frozen with embedded third-party libraries circa 2005–2012: zlib, SQLite, FreeType, lcms, libwebp, Hunspell, and the list goes on. Updating them isn't strictly necessary on principle, but we're not here to use this browser; we're here to look under the hood.

Just blindly updating the code doesn't always work, though for some libraries, it was mundane or required only minor fixes. **FreeType** updated from 2.4.8 to 2.13.3 without regressions; everything matched, right down to the fact that two "crashing" tests crash exactly the same way on the old and new versions. **libwebp** updated from 0.2.0 to 1.6.0, and here I actually had to regenerate the test assets because the new renderer drew things better. A similar story with lcms—the defaults for handling the alpha channel changed.

Some third-party libraries contained specific patches. For instance, in **zlib**, Opera implanted the concept of data "classes" into deflate—their defense against the CRIME attack for compressing SPDY headers (cookies are compressed separately from the rest so that the size of the compressed output doesn't leak the secret). When updating to 1.3.1, this patch had to be ported to modern deflate and covered with tests under sanitizers. A bonus discovery: the old fork had a latent configuration bug causing **all compression levels to degrade to the weakest one**—it's highly likely nobody at Opera ever knew their `deflate-9` was actually `deflate-1`.

**SQLite** was also modified: Opera turned the progress handler into a cooperative scheduler—the query virtual machine could pause mid-sentence and **resume the exact same query from the same opcode** (stock SQLite kills the query upon interruption). Porting this patch from VDBE 3.7.9 to 3.53.3 was its own adventure involving an off-by-one error at the resumption point. Testing also revealed that modern SQLite bumped the page size from 1K to 4K, which promptly blew up the Web SQL quotas that were hardcoded for kilobyte-sized pages.

At the time of writing, only Hunspell remains untouched: the library completely changed its operating principles and API, and its integration is so invasive (a tremendous rarity in Opera's code, but completely justified here) that I haven't quite mustered the courage to touch it yet.

### V.III The Death of Pike and Other Adventures

Concurrently with all this, the toolchain was being dragged into the modern era. The easiest part was porting Flower from Python 2.7 to 3.13. It was purely mechanical work, with `2to3` doing most of the heavy lifting.

Replacing Pike with that same Python took significantly more effort. I deemed replacing it essential—it's too exotic a language, and there is zero reason to keep it around today. The old version of Pike was run in a Docker container, and the entire test corpus was pumped through it to generate baselines. Then, the Python port was compared byte-for-byte against the baselines—even replicating the bugs of the original (Pike stripped trailing spaces in one template and interned dead strings—the port does the same, because those bytes end up in compressed string tables). The result: 98.3% of the corpus files produce byte-for-byte identical tables, all 1,354 files compile, and as modules were turned on, tests started going green by the thousands: `util` — 530/530, `DOM` — 2474/2479, standard library, unicode, formats...

The failing tests proved to be the most interesting.

- **RC4 doesn't work** under OpenSSL 3. RFC 7465 from 2015 forbids its use in TLS.
- The `url.headerreduction` test **has been broken since the Opera days**: the test tables capture global `char*` pointers in the constructor before they are initialized. The port reproduced Pike's behavior bit-for-bit—proving that the test never passed for them either.
- Inline `<svg>` in Presto **never participated in inline layout**—adjacent SVGs stack vertically in defiance of CSS. Apparently, this is another test that was never actually executed.
- Five "crashes" were infrastructure failures: dead test servers, missing fonts, container timezone issues—the test expected to be run in Oslo.

Tests are a boring topic until they start telling stories like this.

Next, it was time to touch the C++. Opera's code was written to **C++98 / C++03** standards and contained a mountain of homegrown wheels, including macros simulating features that later made it into the standard. Building it under modern GCC threw over six thousand warnings. After ripping out **libopeay** and updating the internal forks, things quieted down a bit.

Then came the meticulous work: first, the language level was bumped to C++17 without `-fpermissive`, then to C++20. Zero warnings on GCC/MSVC at `/W4` and `/permissive-`, `register` and `throw()` were excised. The code is almost "C++23-clean"—during a test build against the new standard, only a handful of files break, and all for one incredibly elegant reason that I can't help but share.

`OpString` has a "stealing" copy constructor: it takes a *non-const* reference and **takes ownership** of the source's buffer. Move semantics written in 2003, eight years before C++11. The problem is that C++23 (`P2266`) became much more aggressive about returning local variables as rvalues—and the ancient constructor, demanding an lvalue, stopped binding.

And here arose the primary philosophical question of any restorer: should I rewrite all of this in "normal" modern C++ — `std::string`, `std::unique_ptr`, exceptions? Technically, I could. But I decided: **no**. That original manifesto—"RTTI, STL, namespaces, exceptions: we don't need them"—still stands. Not out of nostalgia, but for the sake of identity: Presto with `std::vector` inside is no longer Presto. The very fabric of this code—the OOM discipline, the custom containers, the custom libc—is what I want to preserve.

The practical compromise I settled on: **use the capabilities of the language, do not use the standard library**. The "homegrown wheels" are modernized in place: `OpAutoPtr` became move-only (with copy `= delete`), and they got their own `op_move`—three lines of code, identical to `std::move` without pulling in a single standard header. By the way, a naive attempt to replace `OpAutoPtr` with `std::unique_ptr` instantly crashed the browser on startup, perfectly illustrating the cost of such replacements: calling `reset(same_pointer)` on Opera's pointer is a no-op; on the standard one, it frees the object and leaves a dangling pointer.

### V.IV Museum of Dead Features

Opera had more dead, experimental, and just plain crazy features than you could shake a stick at. They clearly loved experimenting in Oslo—start counting:

- The M2 mail client;
- An IRC client;
- An RSS reader;
- An internal BitTorrent client;
- Opera Unite—a kind of attempt to create a decentralized web;
- Opera Mini—pages were dynamically executed on the server, and clients received a "pre-chewed" result;
- Opera Turbo—a compressing proxy;
- SPDY—the once-progressive, now-dead ancestor of HTTP/2;
- Numerous integrations with now-dead services, like geolocation or security checks;
- A remote debugger;

I'm sure I forgot something, but you get the point. Today, a significant chunk of this simply will not work. Some things can be restored; others make no sense to revive.

But these are pieces of history. Throwing them away feels wrong; fortunately, Opera was built in a way that allows you to simply hide non-working features behind feature flags. The useless code stays there, mothballed, but always ready.

Except for the macOS code. That was a port to Mac OS X (Cocoa with remnants of Carbon and rudiments of the PowerPC era), which is unbuildable in modern Xcode—I ripped it out without a twinge of conscience.

## VI. What's Next?

Current status: Presto builds on Debian 13/GCC 14 and Visual Studio 2026, runs on Linux x64, Windows x86, and Windows x64, opens modern HTTPS sites via OpenSSL 3 with TLS 1.3, has survived eight dependency updates, and keeps thousands of its own tests green. For a project that started with "will this thing even compile," that's not bad.<br />
But we're absolutely not talking about using it for the modern web—at best, we'll see broken layouts and non-functional content on websites. A significant portion of resources won't render at all. Presto is frozen at a 2013 level—ES5.1 with no `Promise`s or arrow functions, CSS with no `grid` or `custom properties`, and HTTP without version 2.

I don't want to jump the gun on whether an old Opera can be taught new tricks. We'll wait and see.

I cannot publish the compiled binaries or the source code (except maybe as patches, though I doubt anyone would want to bother with those). However, I'm open to suggestions and currently thinking about what can be done to actually release the project.

Also, there will be no links here to a Telegram channel, a Patreon, or anything else. That's not the point.

The point is that Opera was a browser built with love and understanding. You can see it in every line of code, from the "we don't need the STL" manifesto to the double rainbow in the CSS grammar. The refusal to publish the source code felt like an injustice back then—and judging by the fact that someone eventually leaked it anyway, the people inside the company felt the same way. Surely some of those people wished a deep dive like this would happen.

Well, now it has.

---

<nav aria-label="Bottom navigation">
  <a href="#top">↑ Back to top</a> · <a href="{{ '/revival/' | relative_url }}" rel="next">Next: The Engine We're Bringing Back to Life →</a> · WEEK: <strong>I</strong>—<a href="{{ '/revival/' | relative_url }}" title="The Engine We're Bringing Back to Life">II</a>—<a href="{{ '/modern/' | relative_url }}" title="The Engine That Learns">III</a>—<a href="{{ '/css/' | relative_url }}" title="The Engine That Needs CSS">IV</a>—<a href="{{ '/es2015-gstreamer/' | relative_url }}" title="The Engine That Outpaced Chrome">V</a>—<a href="{{ '/dragonfly-shadow-dom/' | relative_url }}" title="Opera Dragonfly and Shadow DOM">VI</a> · <strong>English</strong> · <a href="{{ '/ru/' | relative_url }}" lang="ru">Русский</a>
</nav>
