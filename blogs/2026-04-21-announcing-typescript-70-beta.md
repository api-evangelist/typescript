---
title: "Announcing TypeScript 7.0 Beta"
url: "https://devblogs.microsoft.com/typescript/announcing-typescript-7-0-beta/"
date: "Tue, 21 Apr 2026 18:24:17 +0000"
author: "Daniel Rosenwasser"
feed_url: "https://devblogs.microsoft.com/typescript/feed/"
---
<p>Today we are absolutely thrilled to announce the release of TypeScript 7.0 Beta!</p>
<p>If you haven&#8217;t been following TypeScript 7.0&#8217;s development, this release is significant in that it is built on a completely new foundation.
Over the past year, we have been porting the existing TypeScript codebase from TypeScript (as a bootstrapped codebase that compiles to JavaScript) over to Go.
With a combination of native code speed and shared memory parallelism, <strong>TypeScript 7.0 is often about 10 times faster</strong> than TypeScript 6.0.</p>
<p>Don&#8217;t let the &#8220;beta&#8221; label fool you &#8211; you can probably start using this in your day-to-day work immediately.
The new Go codebase was methodically ported from our existing implementation rather than rewritten from scratch, and its type-checking logic is structurally identical to TypeScript 6.0.
This architectural parity ensures the compiler continues to enforce the exact same semantics you already rely on.
TypeScript 7.0 has been evaluated against the enormous test suite we&#8217;ve built up over the span of a decade, and is already in use in multiple multi-million line-of-code codebases both inside and outside Microsoft.
It is highly stable, highly compatible, and ready to be put to the test in your daily workflows and CI pipelines <em>today</em>.</p>
<p>For over a year we&#8217;ve been working with many internal Microsoft teams, along with teams at companies like Bloomberg, Canva, Figma, Google, Lattice, Linear, Miro, Notion, Slack, Vanta, Vercel, VoidZero, and more to try out pre-release builds of TypeScript 7.0 on their codebases.
The feedback has been overwhelmingly positive, with many teams reporting similar speedups, shaving off a majority of their build times, and enjoying a much more lightweight and fluid editing experience.
In turn, we feel confident that the beta is in great shape, and we can&#8217;t wait for you to try it out soon.</p>
<h2 id="using-typescript-70-beta">Using TypeScript 7.0 Beta</h2>
<p>To get TypeScript 7.0 Beta, you can install it via npm:</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npm install -D @typescript/native-preview@beta
</code></pre>
<blockquote><p>Note: the package name will eventually be <code>typescript</code> in a future release.</p></blockquote>
<p>From there, you can run <code>tsgo</code> in place of the <code>tsc</code> executable.</p>
<pre class="language-text" style="padding: 10px; border-radius: 10px;"><code>&gt; npx tsgo --version
Version 7.0.0-beta
</code></pre>
<p>The <code>tsgo</code> executable has the same behavior on all TypeScript code as <code>tsc</code> from TypeScript 6.0 &#8211; just much faster.</p>
<p>To try out the editing experience, you can install the <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">TypeScript Native Preview extension for VS Code</a>.
The editor support is rock-solid, and has been widely used by many teams for months now.
It&#8217;s an easy low-friction way to try TypeScript 7.0 out on your codebase immediately.
It uses the same foundation as the command line experience, so you get the same performance improvements in your editor as you do on the command line.
Notably, it&#8217;s also built on the language server protocol, making it easy to run in most modern editors or even tools like Copilot CLI.</p>
<h2 id="running-side-by-side-with-typescript-60">Running Side-by-Side with TypeScript 6.0</h2>
<p>To help you transition from TypeScript 6.0 to TypeScript 7.0, this beta release is available through the <code>@typescript/native-preview</code> package name using the <code>tsgo</code> entry point.
This enables easy validation and comparison between <code>tsc</code> and <code>tsgo</code>.</p>
<p>However, as we mentioned above, the stable release of TypeScript 7.0 will be published under the <code>typescript</code> package and will use the <code>tsc</code> entry point.</p>
<p>Additionally, even though 7.0 Beta is close to production-ready, we won&#8217;t have a stable programmatic API available until at least several months from now with TypeScript 7.1.
Given this, we have made it a priority to ensure TypeScript can be run side-by-side with TypeScript 6.0 for the foreseeable future without any conflicts around &#8220;which <code>tsc</code> is which?&#8221;</p>
<p>As part of the 6.0/7.0 transition process, we&#8217;ve published a new compatibility package, <code>@typescript/typescript6</code>.
This package exposes a new entry point <code>tsc6</code>, so that (if needed) you can run the next release of TypeScript 7.0 (which will provide a <code>tsc</code> binary) side-by-side without naming conflicts.
It will also re-export the TypeScript 6.0 API, so that you can use <code>tsc</code> for TypeScript 7, while other tooling can continue to rely on 6.0.</p>
<p>Because some tools like typescript-eslint expect to import from <code>typescript</code> directly via peer dependencies, we recommend achieving this via npm aliases.
You should be able to run the following command</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npm install -D typescript@npm:@typescript/typescript6
</code></pre>
<p>or modify your <code>package.json</code> as follows:</p>
<pre class="prettyprint language-json" style="padding: 10px; border-radius: 10px;"><code>{
  "devDependencies": {
    "typescript": "npm:@typescript/typescript6@^6.0.0",
  }
}
</code></pre>
<p>In the future we will have more specific guidance for using a TS7-powered <code>tsc</code> alongside a TS6-powered <code>tsc6</code>.</p>
<p><!-- Note that doing this will leave you only with a `tsc6` executable. When the time comes to transition to TypeScript 7.0, you can add another alias for TypeScript 7 and `npx tsc` will just work with 7.0: ```json5 // NOTE, WILL NOT WORK *YET*. { "devDependencies": { "typescript": "npm:@typescript/typescript6@~6.0.3", "typescript-7": "npm:typescript@^7.0.0", // not yet a valid package! } } ``` --></p>
<h2 id="parallelization-and-controls">Parallelization and Controls</h2>
<p>TypeScript 7.0 now performs many steps in parallel, including parsing, type-checking, and emitting.
Some of these steps, like parsing and emitting can mostly be done independently across files.
As such, parallelization automatically scales well with larger codebases with relatively little overhead.
But not every step in a TypeScript build is easily parallelizable.</p>
<h3 id="checker-parallelization">Checker Parallelization</h3>
<p>Other steps, like type-checking, have more complex dependencies across files.
Most files end up relying on the same type information from their dependencies and the global scope, and so running type-checkers completely independently would be wasteful &#8211; both in computation and memory.
On the other hand, type-checking occasionally relies on the relative ordering of information in a program, and so type-checking from scratch must always check the same files in an identical order to ensure the same results.</p>
<p>To enable parallelization while avoiding these pitfalls, TypeScript 7.0 creates a fixed number of type-checker workers with their own view of the world.
These type-checking workers may end up duplicating some common work, but given the same input files, they will always divide them identically and produce the same results.</p>
<p>The default number of type-checking workers is 4, but it can be configured with the new <code>--checkers</code> flag.
You may find that increasing this number can further speed up builds on larger codebases where typical machines have more CPU cores, but will typically come at the cost of increased memory usage.
Likewise, machines with fewer CPU cores (e.g. CI runners) may want to decrease this number to avoid unnecessary overhead.</p>
<p>In rare cases, varying the number of <code>--checkers</code> may surface order-dependent results.
Specifying a fixed number of checkers across your team can help ensure everyone is getting the same results, but is up to the discretion of each team.</p>
<h3 id="project-reference-builder-parallelization">Project Reference Builder Parallelization</h3>
<p>TypeScript 7.0 can parallelize builds within a project, but it can now also build multiple projects at once as well.
This behavior can be configured with the new <code>--builders</code> flag, which controls the number of parallel project reference builders that can run at once.
This can be particularly helpful for monorepos with many projects.</p>
<p>Like <code>--checkers</code>, increasing the number of builders can speed up builds, but may come at the cost of increased memory usage.
It also has a multiplicative effect with <code>--checkers</code>, so it&#8217;s important to find the right balance for your machine and codebase.
For example, building with <code>--checkers 4 --builders 4</code> allows up to 16 type-checkers to run at once, which may be excessive.</p>
<p>Unlike <code>--checkers</code>, varying the number of builders should not produce different results;
however, building project references is fundamentally bottlenecked by the dependency graph of projects (with the exception of type-checking on codebases that leverage <code>--isolatedDeclarations</code> and separate syntactic declaration file emit).</p>
<h3 id="single-threaded-mode">Single-Threaded Mode</h3>
<p>In some cases, it can be helpful to enforce single-threaded operation throughout the compiler.
This may be useful for debugging, comparing performance with TypeScript 6 and 7, when orchestrating parallel builds externally, or for running in environments with very limited resources.
To enable single-threaded mode, you can use the new <code>--singleThreaded</code> flag.
This will not only cap the number of type-checking workers to 1, but also ensure parsing and emitting are done in a single thread.</p>
<h2 id="updates-since-5x-and-new-behaviors-from-60">Updates Since 5.x, and New Behaviors from 6.0</h2>
<p>TypeScript 7.0 is made to be compatible with TypeScript 6.0&#8217;s type-checking and command-line behavior.
Any TypeScript code that compiles cleanly with TypeScript 6.0 (with the <code>stableTypeOrdering</code> flag on, and without the <code>ignoreDeprecations</code> flag set) should compile identically in TypeScript 7.0.</p>
<p>With that said, TypeScript 7.0 adopts 6.0&#8217;s new defaults, and provides hard errors in the face of any flags and constructs deprecated in TypeScript 6.0.
This is notable as 6.0 is still relatively new, and many projects will need to adapt to its new behaviors.
We encourage developers to adopt TypeScript 6.0 to make the transition to TypeScript 7.0 easier, and you can also read <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-6-0/">the TypeScript 6.0 release blog post</a> for more details on these deprecations.</p>
<p>At a glance, the notable default changes to configuration are:</p>
<ul>
<li><code>strict</code> is <code>true</code> by default.</li>
<li><code>module</code> defaults to <code>esnext</code>.</li>
<li><code>target</code> defaults to the current stable ECMAScript version immediately preceding <code>esnext</code>.</li>
<li><code>noUncheckedSideEffectImports</code> is <code>true</code> by default.</li>
<li><code>libReplacement</code> is <code>false</code> by default.</li>
<li><code>stableTypeOrdering</code> is <code>true</code> by default, and cannot be turned off.</li>
<li><code>rootDir</code> now defaults to <code>./</code>, and inner source directories must be explicitly set.</li>
<li><code>types</code> now defaults to <code>[]</code>, and the old behavior can be restored by setting it to <code>["*"]</code>.</li>
</ul>
<p>We believe the <code>rootDir</code> and <code>types</code> changes may be the most &#8220;surprising&#8221; changes, but they can be mitigated easily.
Projects where the <code>tsconfig.json</code> sits outside of a directory like <code>src</code> will simply need to include <code>rootDir</code> to preserve the same directory structure.</p>
<pre class="prettyprint language-diff" style="padding: 10px; border-radius: 10px;"><code>  {
      "compilerOptions": {
          // ...
+         "rootDir": "./src"
      },
      "include": ["./src"]
  }
</code></pre>
<p>For the <code>types</code> change, projects that depend on specific global declarations will need to list them explicitly. For example,</p>
<pre class="prettyprint language-diff" style="padding: 10px; border-radius: 10px;"><code>  {
      "compilerOptions": {
          // Explicitly list the @types packages you need (e.g. bun, mocha, jasmine, etc.)
+         "types": ["node", "jest"]
      }
  }
</code></pre>
<p>The deprecations that have turned into hard errors with no-op behavior are:</p>
<ul>
<li><code>target: es5</code> is no longer supported.</li>
<li><code>downlevelIteration</code> is no longer supported.</li>
<li><code>moduleResolution: node/node10</code> are no longer supported, with <code>nodenext</code> and <code>bundler</code> being recommended instead.</li>
<li><code>module: amd, umd, systemjs, none</code> are no longer supported, with <code>esnext</code> or <code>preserve</code> being recommended in conjunction with bundlers or browser-based module resolution.</li>
<li><code>baseUrl</code> is no longer supported, and <code>paths</code> can be updated to be relative to the project root instead of <code>baseUrl</code>.</li>
<li><code>moduleResolution: classic</code> is no longer supported, and <code>bundler</code> or <code>nodenext</code> are the recommended replacements.</li>
<li><code>esModuleInterop</code> and <code>allowSyntheticDefaultImports</code> cannot be set to <code>false</code>.</li>
<li><code>alwaysStrict</code> is assumed to be <code>true</code> and can no longer be set to <code>false</code></li>
<li>The <code>module</code> keyword cannot be used in namespace declarations.</li>
<li>The <code>asserts</code> keyword cannot be used on imports, and must use the <code>with</code> keyword instead (to align with developments on ECMAScript&#8217;s import attribute syntax).</li>
<li><code>/// &lt;reference no-default-lib /&gt;</code> directives are no longer respected under <code>skipDefaultLibCheck</code>.</li>
<li>Command line builds cannot take file paths when the current directory contains a <code>tsconfig.json</code> file unless passed an explicit <code>--ignoreConfig</code> flag.</li>
</ul>
<h3 id="javascript-differences">JavaScript Differences</h3>
<p>As we ported the existing codebase, we also took the opportunity to revisit how our JavaScript support works.</p>
<p>TypeScript originally supported JavaScript files by using JSDoc comments and recognizing certain code patterns for analysis and type inference.
Lots of the time, this was based on popular coding patterns, but occasionally it was based on whatever people <em>might</em> be writing that Closure and the JSDoc doc generating tool might understand.
While this approach was helpful for developers with loosely-written JSDoc codebases, it required a number of compromises and special cases to work well, and diverged in a number of ways from TypeScript&#8217;s analysis in <code>.ts</code> files.</p>
<p>In TypeScript 7.0, we have reworked our JavaScript support to be more consistent with how we analyze TypeScript files.
Some of the differences include:</p>
<ul>
<li>Values cannot be used where types are expected &#8211; instead, write <code>typeof someValue</code></li>
<li><code>@enum</code> is not specially recognized anymore &#8211; create a <code>@typedef</code> on <code>(typeof YourEnumDeclaration)[keyof typeof YourEnumDeclaration]</code>.</li>
<li>A standalone <code>?</code> is no longer usable as a type &#8211; use <code>any</code> instead.</li>
<li><code>@class</code> does not make a function a constructor &#8211; use a <code>class</code> declaration instead.</li>
<li>Postfix <code>!</code> is not supported &#8211; just use <code>T</code>.</li>
<li>Type names must be defined within a <code>@typedef</code> tag (i.e. <code>/** @typedef {T} TypeAliasName */</code>), not adjacent to an identifier (i.e. <code>/** @typedef {T} */ TypeAliasName;</code>).</li>
<li>Closure-style function syntax (e.g. <code>function(string): void</code>) is no longer supported &#8211; use TypeScript shorthands instead (e.g. <code>(s: string) =&gt; void</code>).</li>
</ul>
<p>Additionally, some JavaScript patterns, like aliasing <code>this</code> and reassigning the entirety of a function&#8217;s <code>prototype</code> are no longer specially treated.</p>
<p>While some of our JS support is in flux, we have been updating this <a href="https://github.com/microsoft/typescript-go/blob/main/CHANGES.md"><code>CHANGES.md</code> file</a> to capture the differences between TypeScript 6.0 and 7.0 in more detail.</p>
<h2 id="editor-experience">Editor Experience</h2>
<p>TypeScript 7.0&#8217;s performance improvements are not limited to the command line experience &#8211; they also extend to the editor experience too.
The <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">TypeScript Native Preview extension for VS Code</a> provides a seamless way to try out TypeScript 7.0 in your editor, and has seen widespread use.</p>
<p>Since it first debuted, we&#8217;ve added in missing functionality like auto-imports, expandable hovers, inlay hints, code lenses, go-to-source-definition, JSX linked editing and tag completions, and more.
Additionally, we&#8217;ve rebuilt much of our testing and diagnostics infrastructure to make sure the quality bar is high.</p>
<p>This extension respects most of the same configuration settings as the built-in TypeScript extension for Visual Studio Code, along with most of the same features.
While a few things are still coming (like semantics-enhanced highlighting, more-specific import management commands, etc.), the extension is already powerful, stable, and fast.</p>
<h2 id="upcoming-work">Upcoming Work</h2>
<p>In the coming weeks, we expect to ship a more efficient implementation of <code>--watch</code>, and meet parity on declaration file emit from JavaScript files.
We will also be working on minor editor feature gaps like &#8220;find file references&#8221; from the file explorer, and surfacing the more granular &#8220;sort imports&#8221; and &#8220;remove unused imports&#8221; commands instead of just the more general &#8220;organize imports&#8221; command.</p>
<p>Beyond this, we&#8217;ll be developing a stable programmatic API for TypeScript 7.1 or later, improving our real-world testing infrastructure, and addressing feedback.</p>
<h2 id="the-road-to-typescript-70">The Road to TypeScript 7.0</h2>
<p>With TypeScript 7.0 Beta now available, the team is focusing on bug fixes, compatibility work, editor polish, and performance improvements as we move toward a stable release.
Our current plan is to release TypeScript 7.0 within the next two months, with a release candidate available a few weeks prior.
The release candidate will be the point where we expect TypeScript 7&#8217;s behavior to be finalized, with changes after that focused on critical fixes to regressions.</p>
<p>Between now and then, we would especially appreciate feedback from trying TypeScript 7.0 on real projects.
If you run into any issues, please let us know on <a href="https://github.com/microsoft/typescript-go/issues">the issue tracker for microsoft/typescript-go</a> so we can make sure the stable release is in great shape.</p>
<p>We also encourage you to share your experience using TypeScript 7.0 and tag <a href="https://bsky.app/profile/typescriptlang.org">@typescriptlang.org on Bluesky</a> or <a href="https://fosstodon.org/@TypeScript/">@typescript@fosstodon.org on Mastodon</a>, or <a href="https://twitter.com/typescript">@typescript on Twitter</a>.</p>
<p>Our team is incredibly excited for you to try this release out, so try it today and let us know what you think. Happy hacking!</p>
<p>&#8211; The TypeScript Team</p>
<p>The post <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-7-0-beta/">Announcing TypeScript 7.0 Beta</a> appeared first on <a href="https://devblogs.microsoft.com/typescript">TypeScript</a>.</p>
