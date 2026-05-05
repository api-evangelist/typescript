---
title: "Progress on TypeScript 7 – December 2025"
url: "https://devblogs.microsoft.com/typescript/progress-on-typescript-7-december-2025/"
date: "Tue, 02 Dec 2025 17:31:32 +0000"
author: "Daniel Rosenwasser"
feed_url: "https://devblogs.microsoft.com/typescript/feed/"
---
<p>Earlier this year, <a href="https://devblogs.microsoft.com/typescript/typescript-native-port/">the TypeScript team announced that we&#8217;ve been porting the compiler and language service to native code</a> to take advantage of better raw performance, memory usage, and parallelism.
This effort (codenamed &#8220;Project Corsa&#8221;, and soon &#8220;TypeScript 7.0&#8221;) has been a significant undertaking, but we&#8217;ve made big strides in the past few months.
We&#8217;re excited to give some updates on where we are, and show you how &#8220;real&#8221; the new TypeScript toolset is today.</p>
<p>We also have news about our upcoming roadmap, and how we&#8217;re prioritizing work on TypeScript 7.0 to drive our port to completion.</p>
<h2 id="editor-support-and-language-service">Editor Support and Language Service</h2>
<p>For a lot of developers, a project rewrite might feel entirely theoretical until it&#8217;s finally released.
That&#8217;s not the case here.</p>
<p><strong>TypeScript&#8217;s native previews are fast, stable, and easy to use today &#8211; including in your editor</strong>.</p>
<p>TypeScript&#8217;s language service (the thing that powers your editor&#8217;s TypeScript and JavaScript features) is also a core part of the native port effort, and is easy to try out.
You can grab the latest version from the <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">Visual Studio Code Marketplace</a> which gets updated every day.</p>
<p>Our team is still porting features and fixing minor bugs, but most of what really makes the existing TypeScript editing experience is there and working well.</p>
<p>That includes:</p>
<ul>
<li>Code Completions (<em>including auto-imports!</em>)</li>
<li>Go-to-Definition</li>
<li>Go-to-Type-Definition</li>
<li>Go-to-Implementation</li>
<li>Find-All-References</li>
<li>Rename</li>
<li>Quick Info/Hover Tooltips</li>
<li>Signature Help</li>
<li>Formatting</li>
<li>Selection Ranges</li>
<li>Code Lenses</li>
<li>Call Hierarchy</li>
<li>Document Symbols</li>
<li>Quick Fixes for Missing Imports</li>
</ul>
<p>You might notice a few things that stand out since our last major update &#8211; auto-imports, find-all-references, rename, and more.
We know that these features were the missing pieces that held a lot of developers back from trying out the native previews.
We&#8217;re happy to say that these are now reimplemented and ready for day-to-day use!
These operations now work in any TypeScript or JavaScript codebase &#8211; including those with project references.</p>
<p>We&#8217;ve also rearchitected parts of our language service to improve reliability while also leveraging shared-memory parallelism.
While some teams reported the original experience was a bit &#8220;crashy&#8221; at times, they often put up with it because of the speed improvements.
The new architecture is more robust, and should be able to handle codebases, both big and small, without issues.</p>
<p>While there is certainly more to port and polish, your team will likely find that trying out TypeScript&#8217;s native previews is worth it.
You can expect faster load times, less memory usage, and a more snappy/responsive editor on the whole.</p>
<p>If you&#8217;re ever unhappy with the experience, our extension makes it easy to toggle between VS Code&#8217;s built-in TypeScript experience and the new one.
We really encourage you and your team to <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">try out the native preview extension for VS Code today</a>!</p>
<h2 id="compiler">Compiler</h2>
<p>The TypeScript compiler has also made significant progress in the native port.
Just like our VS Code extension, <a href="https://www.npmjs.com/package/@typescript/native-preview">we have been publishing nightly preview builds of the new compiler</a> under the package name <code>@typescript/native-preview</code>.
You can install it via npm like so:</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code># local dev dependency
npm install -D @typescript/native-preview

# global install
npm install -g @typescript/native-preview
</code></pre>
<p>This package provides a <code>tsgo</code> command that works similarly to the existing <code>tsc</code> command.
The two can be run side-by-side.</p>
<p>A frequent question we get is whether it&#8217;s &#8220;safe&#8221; to use TypeScript 7 to validate a build; in other words, does it reliably find the same errors that TypeScript 5.9 does?</p>
<p>The answer is a resounding yes.
TypeScript 7&#8217;s type-checking is very nearly complete.
For context, we have around 20,000 compiler test cases, of which about 6,000 produce at least one error in TypeScript 6.0. In all but 74 cases, TypeScript 7 also produces at least one error.
Of those remaining 74 cases, all are known incomplete work (e.g. regular expression syntax checking or <code>isolatedDeclarations</code> errors) or are related to known intentional changes (deprecations, default settings changes, etc.).
You can confidently use TypeScript 7 today to type-check your project for errors.</p>
<p>Beyond single-pass/single-project type checking, the command-line compiler has reached major parity as well.
Features like <code>--incremental</code>, project reference support, and <code>--build</code> mode are also now all ported over and working!
This means most projects can now try the native preview with minimal changes.</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code># Running tsc in --build mode...
tsc -b some.tsconfig.json --extendedDiagnostics

# Running the *new compiler* in --build mode...
tsgo -b some.tsconfig.json --extendedDiagnostics
</code></pre>
<p>Not only are these features now available, they should be <em>dramatically</em> faster than the existing versions implemented in TypeScript 5.9 and older (a.k.a. the &#8220;Strada codebase&#8221;).
<a href="https://devblogs.microsoft.com/typescript/typescript-native-port/">As we&#8217;ve described previously</a>, this comes in part from native code performance, but also from the use of shared-memory parallelism.
More specifically what this means is that not only can TypeScript now do fast multi-threaded builds on single projects;
it can now build up multiple projects in parallel as well!
Combined with our reimplementation of <code>--incremental</code>, we&#8217;re close to making TypeScript builds feel instantaneous for smaller changes in large projects.</p>
<p>Just as a reminder, even without <code>--incremental</code>, TypeScript 7 often sees close to a 10x speedup over the 6.0 compiler on full builds!</p>
<table>
<thead>
<tr>
<th>Project</th>
<th>tsc (6.0)</th>
<th>tsgo (7.0)</th>
<th>Delta</th>
<th>Speedup Factor</th>
</tr>
</thead>
<tbody>
<tr>
<td><strong>sentry</strong></td>
<td>133.08s</td>
<td>16.25s</td>
<td>116.84s</td>
<td>8.19x</td>
</tr>
<tr>
<td><strong>vscode</strong></td>
<td>89.11s</td>
<td>8.74s</td>
<td>80.37s</td>
<td>10.2x</td>
</tr>
<tr>
<td><strong>typeorm</strong></td>
<td>15.80s</td>
<td>1.06s</td>
<td>14.20s</td>
<td>9.88x</td>
</tr>
<tr>
<td><strong>playwright</strong></td>
<td>9.30s</td>
<td>1.24s</td>
<td>8.07s</td>
<td>7.51x</td>
</tr>
</tbody>
</table>
<h2 id="expected-differences-from-typescript-59">Expected Differences from TypeScript 5.9</h2>
<p>There are some caveats to using the new compiler that we want to call out.
Many of these are point-in-time issues that we plan to resolve before the final 7.0 release, but some are driven more by long-term decisions to make the default TypeScript experience better.
The promise of TypeScript 7.0 means that we will need to heavily shift our focus to the new codebase to close existing gaps and put the new toolchain in the hands of more developers.
But let&#8217;s first dive in and cover some of the current changes and limitations.</p>
<h3 id="deprecation-compatibility">Deprecation Compatibility</h3>
<p>TypeScript 7.0 will remove behaviors and flags that we plan to deprecate in TypeScript 6.0.
Right now, you can see <a href="https://github.com/microsoft/TypeScript/issues?q=milestone%3A%22TypeScript%206.0.0%22%20label%3A%22Breaking%20Change%22">the list of upcoming deprecations in 6.0 on our issue tracker</a>.
Some prominent examples include:</p>
<ul>
<li><a href="https://github.com/microsoft/TypeScript/issues/62333"><code>--strict</code> will be enabled by default</a></li>
<li><a href="https://github.com/microsoft/TypeScript/issues/62198"><code>--target</code> will default to the latest stable ECMAScript target (e.g. <code>es2025</code>)</a></li>
<li><a href="https://github.com/microsoft/TypeScript/issues/62196"><code>--target es5</code> will be removed, with <code>es2015</code> being the lowest-supported target</a></li>
<li><a href="https://github.com/microsoft/TypeScript/issues/62207"><code>--baseUrl</code> will be removed</a></li>
<li><a href="https://github.com/microsoft/TypeScript/issues/62200"><code>--moduleResolution node10</code> (a.k.a. <code>node</code>) will be removed in favor of <code>bundler</code> and <code>nodenext</code></a></li>
<li><a href="https://github.com/microsoft/TypeScript/issues/62194"><code>rootDir</code> defaults to the current directory, and using <code>outDir</code> either requires an explicit <code>rootDir</code> or for top-level source files to be in the same directory as the <code>tsconfig.json</code></a></li>
</ul>
<p>This is not comprehensive, so <a href="https://github.com/microsoft/TypeScript/issues?q=milestone%3A%22TypeScript%206.0.0%22%20label%3A%22Breaking%20Change%22">check out the issue tracker</a> for the current state of things.
If your project relies on any of these deprecated behaviors, you may need to make some changes to your codebase or <code>tsconfig.json</code> to ensure compatibility with TypeScript 7.0.</p>
<p>Our team <a href="https://github.com/andrewbranch/ts5to6">has been experimenting with a tool called <code>ts5to6</code> to help update your <code>tsconfig.json</code> automatically</a>.
The tool uses heuristics on <code>extends</code> and <code>references</code> to help update other projects in your codebase.
Currently it can only update the <code>baseUrl</code> and <code>rootDir</code> settings, but more may be added in the future.</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npx @andrewbranch/ts5to6 --fixBaseUrl your-tsconfig-file-here.json
npx @andrewbranch/ts5to6 --fixRootDir your-tsconfig-file-here.json
</code></pre>
<h3 id="emit---watch-and-api">Emit, <code>--watch</code>, and API</h3>
<p>Even with 6.0-readiness, there are some circumstances in which the new compiler can&#8217;t immediately be swapped in.</p>
<p>For one, the JavaScript emit pipeline is not entirely complete.
If you <em>don&#8217;t</em> need JavaScript emit from TypeScript (e.g. if you use Babel, esbuild, or something else), or if you are targeting modern browsers/runtimes, running <code>tsgo</code> for your build will work just fine.
But if you rely on the TypeScript to target older runtimes, our support for downlevel compilation realistically only goes as far back the <code>es2021</code> target, and with no support for compiling decorators.
We plan to address this with full <code>--target</code> support going back to <code>es2015</code>, but that work is still ongoing.</p>
<p>Another issue is that our new <code>--watch</code> mode may be less-efficient than the existing TypeScript compiler in some scenarios.
In some cases you can find other solutions like running <a href="https://www.npmjs.com/package/nodemon">nodemon</a> and <code>tsgo</code> with the <code>--incremental</code> flag.</p>
<p>Finally, Corsa/TypeScript 7.0 will not support the existing Strada API.
The Corsa API is still a work in progress, and no stable tooling integration exists for it.
That means any tools like linters, formatters, or IDE extensions that rely on the Strada API will not work with Corsa.</p>
<p>The workaround for some of these issues may be to have the <code>typescript</code> and <code>@typescript/native-preview</code> packages installed side-by-side, and use the ≤6.0 API for tooling that needs it, with <code>tsgo</code> for type-checking.</p>
<h3 id="javascript-checking-and-jsdoc-compatibility">JavaScript Checking and JSDoc Compatibility</h3>
<p>Another thing that we want to call out is that our JavaScript type-checking support (partly powered by JSDoc annotations) has been rewritten from the ground up.
In an effort to simplify our internals, we have stripped down some of our support for complex and some less-used patterns that we previously recognized and analyzed.
For example, TypeScript 7.0 does not recognize the <code>@enum</code> and <code>@constructor</code> tags.
We also dropped some &#8220;relaxed&#8221; type-checking rules in JavaScript, such as interpreting:</p>
<ul>
<li><code>Object</code> as <code>any</code>,</li>
<li><code>String</code> as <code>string</code>,</li>
<li><code>Foo</code> as <code>typeof Foo</code> when the latter would have been valid in a TypeScript file,</li>
<li>all <code>any</code>, <code>unknown</code>, and <code>undefined</code>-typed parameters as optional</li>
</ul>
<p>and more.
Some of these are being <a href="https://github.com/microsoft/typescript-go/pull/1426">reviewed and documented here</a>, though the list may need to be updated.</p>
<p>This means that some JavaScript codebases may see more errors than they did before, and may need to be updated to work well with the new compiler.
On the flip side, we believe that the new implementation is more robust and maintainable, and aligns TypeScript&#8217;s JSDoc support with its own type syntax.</p>
<p>If you feel like something <em>should</em> be working or is missing from our JavaScript type-checking support, we encourage you to <a href="https://github.com/microsoft/typescript-go">file an issue on our GitHub repository</a>.</p>
<h2 id="focusing-on-the-future">Focusing on the Future</h2>
<p>When we set out to rewrite TypeScript last year, there were a lot of uncertainties.
Would the community be excited?
How long would it take for the codebase to stabilize?
How quickly could teams adopt this new toolset?
What degree of compatibility would we be able to deliver?</p>
<p>On all fronts, we&#8217;ve been very pleasantly surprised.
We&#8217;ve been able to implement a type-checker with extremely high compatibility.
As a result, projects both inside and outside Microsoft report that they&#8217;ve been able to easily use the native compiler with minimal effort.
Stability is going well, and we&#8217;re on track to finish most language service features by the end of the year.
Many teams are already using Corsa for day-to-day work without any blocking issues.</p>
<p>With 6.0 around the corner, we have to consider what happens next in the JavaScript codebase.
Our initial plan was to continue work in the 6.0 line &#8220;until TypeScript 7+ reaches sufficient maturity and adoption&#8221;.
We know there is still remaining work to do to unblock more developers (e.g. more work on the API surface), and closing down development on the Strada line &#8211; our JavaScript-based compiler &#8211; is the best way for us to get those blockers removed sooner rather than later.
To help us get these done as soon as possible, we&#8217;re taking a few steps in the Strada project.</p>
<h3 id="typescript-60-is-the-last-javascript-based-release">TypeScript 6.0 is the Last JavaScript-Based Release</h3>
<p>TypeScript 6.0 will be our last release based on the existing TypeScript/JavaScript codebase.
In other words, we do not intend to release a TypeScript 6.1, though we may have patch releases (e.g. 6.0.1, 6.0.2) under rarer circumstances.</p>
<p>You can think of TypeScript 6.0 as a &#8220;bridge&#8221; release between TypeScript 5.9 line and 7.0.
6.0 will deprecate features to align with 7.0, and will be highly compatible in terms of type-checking behavior.</p>
<p>Most codebases which need editor-side Strada-specific functionality (e.g. language service plugins) should be able to use 6.0 for editor functionality, and 7.0 for fast command-line builds without much trouble.
The inverse is also true: developers can use 7.0 for a faster experience in their editor, and 6.0 for command-line tooling that relies on the TypeScript 6.0 API.</p>
<p>Additional servicing after TypeScript 6.0 is released will be in the form of patch releases, and will only be issued in the case of:</p>
<ul>
<li>security issues,</li>
<li>high-severity regressions (i.e. new and serious bugs that were not present in 5.9),</li>
<li>high-severity fixes related to 6.0/7.0 compatibility.</li>
</ul>
<p>As with previous releases, patch releases will be infrequent, and only issued when absolutely necessary.</p>
<p>But as for right now, we want to ensure that TypeScript 6.0 and 7.0 are as compatible as possible.
We&#8217;ll be holding a very high bar in terms of which open PRs are merged into the 6.0 line.
That takes effect today, and it means most developers will have to set expectations for which issues will be addressed in TypeScript 6.0.
Additionally, contributors should understand that we are very unlikely to merge pull requests into 6.0, with most of our focus going bringing 7.0 to parity and stability.
We want to be transparent on this front so that there is no &#8220;wasted&#8221; work, and so that our team can avoid complications in porting changes between the two codebases.</p>
<h3 id="resetting-language-service-issues">Resetting Language Service Issues</h3>
<p>While most of the core type-checking code has been ported over without any behavioral differences, the language service is a different story.
Given the new architecture, much of the code that powers completions, hover tooltips, navigation, and more, has been heavily rewritten.
Additionally, TypeScript 7.0 uses the standard LSP protocol instead of the custom TSServer protocol, so some behavior specific to the TypeScript VS Code Extension may have changed.</p>
<p>As a result, any bugs or suggestions specific to language service behavior are likely not to reproduce in the 7.0 line, or need a &#8220;reset&#8221; in the conversation.</p>
<p>These issues are very time-consuming to manually verify, so instead we&#8217;ll be closing existing issues related to language service behavior.
If you run into an issue that was closed under the &#8220;7.0 LS Migration&#8221; label, please <em>log a new issue</em> after validating that it can be reproduced in the native nightly extension.
For functionality that is not yet ported to 7.0, please wait until that functionality is present before raising a new issue.</p>
<h2 id="whats-next">What&#8217;s Next?</h2>
<p>When we <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-native-previews/">unveiled our native previews</a> a few months back, we had to manage expectations on the state of the project.
We&#8217;re now at the point where we can confidently say that the native TypeScript experience is real, stable, and ready for broader use.
But we are absolutely still looking for feedback.</p>
<p>So we encourage you to install the <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">VS Code native preview extension</a>, use the <a href="https://www.npmjs.com/package/@typescript/native-preview"><code>@typescript/native-preview</code> compiler package</a> where you can, and try it out in your projects.
Let us know what you think, and file issues on our <a href="https://github.com/microsoft/typescript-go/issues">GitHub repository</a> to help us fix up any issues and prioritize what to work on next.</p>
<p>We&#8217;re excited about the future of TypeScript, and we can&#8217;t wait to get TypeScript 7.0 into your hands!</p>
<p>Happy Hacking!</p>
<p>The post <a href="https://devblogs.microsoft.com/typescript/progress-on-typescript-7-december-2025/">Progress on TypeScript 7 &#8211; December 2025</a> appeared first on <a href="https://devblogs.microsoft.com/typescript">TypeScript</a>.</p>
