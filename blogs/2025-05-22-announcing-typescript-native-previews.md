---
title: "Announcing TypeScript Native Previews"
url: "https://devblogs.microsoft.com/typescript/announcing-typescript-native-previews/"
date: "Thu, 22 May 2025 15:04:21 +0000"
author: "Daniel Rosenwasser"
feed_url: "https://devblogs.microsoft.com/typescript/feed/"
---
<p>This past March <a href="https://devblogs.microsoft.com/typescript/typescript-native-port/">we unveiled our efforts to port the TypeScript compiler and toolset to native code</a>.
This port has achieved a 10x speed-up on most projects &#8211; not just by using a natively-compiled language (Go), but also through using shared memory parallelism and concurrency where we can benefit.
Since then, we have made several strides towards running on large complex real-world projects.</p>
<p>Today, we are excited to announce broad availability of TypeScript Native Previews.
As of today, you will be able to use npm to get a preview of the native TypeScript compiler.
Additionally, you&#8217;ll be able to use a preview version of our editor functionality for <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">VS Code through the Visual Studio Marketplace</a>.</p>
<p>To get the compiler over npm, you can run the following command in your project:</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npm install -D @typescript/native-preview
</code></pre>
<p>This package provides an executable called <code>tsgo</code>.
This executable runs similarly to <code>tsc</code>, the existing executable that the <code>typescript</code> package makes available:</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npx tsgo --project ./src/tsconfig.json
</code></pre>
<p>Eventually we will rename <code>tsgo</code> to <code>tsc</code> and move it to the <code>typescript</code> package.
For now, it lives separately for easier testing.
The new executable is still a work in progress, but is suitable to type-check and build many real-world projects.</p>
<p>But we know that a command-line compiler is only half the story.
We&#8217;ve heard teams are eager to see what the new editing experience is like too, and so you can now install the new &#8220;TypeScript (Native Preview)&#8221; extension in Visual Studio Code.
You can easily install it off of the <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">VS Code Extension Marketplace</a>.</p>
<p>Because the extension is still in early stages of development, it defers to the built-in TypeScript extension in VS Code.
For that reason, the extension will need to be enabled even after installation.
You can do this by opening <a href="https://code.visualstudio.com/docs/getstarted/userinterface#_command-palette">VS Code&#8217;s command palette</a> and running the command &#8220;TypeScript Native Preview: Enable (Experimental)&#8221;.</p>
<p><img alt="TypeScript Native Preview: Enable (Experimental)" src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/05/native-preview-enable-202505.png" /></p>
<p>Alternatively, you can toggle this in your settings UI by configuring &#8220;TypeScript &gt; Experimental: Use Tsgo&#8221;</p>
<p><img alt="&quot;Use tsgo&quot; in the VS Code Settings UI" src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/05/native-preview-use-tsgo-202505-1.png" /></p>
<p>or by adding the following line to your JSON settings:</p>
<pre class="prettyprint language-json" style="padding: 10px; border-radius: 10px;"><code>"typescript.experimental.useTsgo": true,
</code></pre>
<h2 id="updates-release-cadence-and-roadmap">Updates, Release Cadence, and Roadmap</h2>
<p>These previews will eventually become TypeScript 7 and will be published nightly so that you can easily try the latest developments on the TypeScript native port effort.
If you use the VS Code extension, you should get automatic updates by default.
If for whatever reason you find any sort of disruption, we encourage you to <a href="https://github.com/microsoft/typescript-go/issues">file an issue</a> and temporarily disable the new language service with the command &#8220;TypeScript Native Preview: Disable&#8221;:</p>
<p><img alt="TypeScript Native Preview: Disable" src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/05/native-preview-enable-202505.png" /></p>
<p>or by configuring any of the settings mentioned above.</p>
<p>Keep in mind, these native previews are missing lots of functionality that stable versions of TypeScript have today.
That includes command-line functionality like <code>--build</code> (though individual projects can still be built with <code>tsgo</code>), <code>--declaration</code> emit, and certain downlevel emit targets.
Similarly, editor functionality like auto-imports, find-all-references, and rename are still pending implementation.
But we encourage developers to check back frequently, as we&#8217;ll be hard at work on these features!</p>
<h2 id="whats-new">What&#8217;s New?</h2>
<p>Since our initial announcement, we have made some notable strides in type-checking support, testability, editor support, and APIs.
We would also love to give a brief update of what we&#8217;ve accomplished, plus what&#8217;s on the horizon.</p>
<p>Note that while the native preview will eventually be called TypeScript 7, we&#8217;ve casually been referring to it as &#8220;Project Corsa&#8221; in the meantime.
We&#8217;ve also been more explicit in referring to the codebase that makes up TypeScript 5.8 as our current JS-based codebase, or &#8220;Strada&#8221;.
So in our updates, you&#8217;ll see us differentiate the native and stable versions of TypeScript as &#8220;Corsa&#8221; (TS7) and &#8220;Strada&#8221; (TS 5.8).</p>
<h3 id="fuller-type-checking-support">Fuller Type-Checking Support</h3>
<p>The majority of the type-checker has been ported at this time.
That is to say, most projects should see the same errors apart from those affected by some intentional changes (e.g. <a href="https://github.com/microsoft/typescript-go/pull/200">this change to the ordering of types</a>) and some stale definitions in <code>lib.d.ts</code>.
If you see any divergences and differences, we encourage you <a href="https://github.com/microsoft/typescript-go/issues">to file an issue to let us know</a>.</p>
<p>It is worth calling out support for two major type-checking features that have been added since our initial announcement: JSX and JavaScript+JSDoc.</p>
<h3 id="jsx-checking-support">JSX Checking Support</h3>
<p>When developers first got access to the <a href="https://github.com/microsoft/typescript-go/">TypeScript native port</a>, we had to temper expectations.
While type-checking was pretty far along, some constructs were still not fully checked yet.
For many developers, the most notable omission was JSX.
While Corsa was able to parse JSX, it would mostly just pass over JSX expressions when type-checking and note that JSX was not-yet supported.</p>
<p>Since then, we&#8217;ve actually added type-checking support for JSX and we can get a better sense of how fast a real JSX project can be built.
As an example codebase, we looked at the codebase for <a href="https://github.com/getsentry/sentry/">Sentry</a>.
If you run TypeScript 5.8 from the repository root with <code>--extendedDiagnostics --noEmit</code>, you&#8217;ll get something like the following:</p>
<pre class="language-text" style="padding: 10px; border-radius: 10px;"><code>$ tsc -p . --noEmit --extendedDiagnostics
Files:                         9306
Lines of Library:             43159
Lines of Definitions:        352182
Lines of TypeScript:        1113969
Lines of JavaScript:           1106
Lines of JSON:                  304
Lines of Other:                   0
Identifiers:                1956007
Symbols:                    3563371
Types:                       999619
Instantiations:             3675199
Memory used:               3356832K
Assignability cache size:    944737
Identity cache size:          43226
Subtype cache size:          110171
Strict subtype cache size:   430338
I/O Read time:                1.40s
Parse time:                   3.48s
ResolveModule time:           1.88s
ResolveTypeReference time:    0.02s
ResolveLibrary time:          0.01s
Program time:                 7.78s
Bind time:                    1.77s
Check time:                  63.26s
printTime time:               0.00s
Emit time:                    0.00s
Total time:                  72.81s
</code></pre>
<p>That&#8217;s over a minute to type-check this codebase!
Let&#8217;s see how the native port fares with some minimal changes.</p>
<pre class="language-text" style="padding: 10px; border-radius: 10px;"><code>$ tsgo -p . --noEmit --extendedDiagnostics
...
Files:              9292
Lines:           1508361
Identifiers:     1954236
Symbols:         5011565
Types:           1689528
Instantiations:  6524885
Memory used:    3892267K
Memory allocs:  61043466
Parse time:       0.712s
Bind time:        0.133s
Check time:       5.882s
Emit time:        0.012s
Total time:       6.761s
</code></pre>
<p>There are some discrepancies in results, but Corsa brings build times down from over a minute to just under 7 seconds on the same machine.
Your results may vary, but in general we&#8217;ve seen consistent speed-ups of over 10x on this specific example.
You can try introducing an error in existing JSX code and <code>tsgo</code> will catch it.</p>
<p>You can see more over <a href="https://github.com/microsoft/typescript-go/pull/762">at our PR to port JSX type-checking</a>, plus some follow-on support <a href="https://github.com/microsoft/typescript-go/pull/780">for tslib and JSX factory imports</a>.</p>
<h3 id="javascript-checking">JavaScript Checking</h3>
<p><a href="https://www.typescriptlang.org/docs/handbook/type-checking-javascript-files.html">TypeScript supports parsing and type-checking JavaScript files</a>.
Because valid JavaScript/ECMAScript doesn&#8217;t support type-specific syntax like annotations or <code>interface</code> and <code>type</code> declarations, TypeScript looks at JSDoc comments in JS source code for type analysis.</p>
<p>The native previews of TypeScript now also support type-checking JS files.
In developing JS type-checking for Corsa, we revisited our early decisions in our implementation.
JavaScript support was built up in a very organic way and, in turn, analyzed very specific patterns that may no longer be (or may never have been) in widespread use.
In order to simplify the new codebase, JavaScript support has been rewritten rather than ported.
As a result, there may be some constructs that may need to be rewritten, or to use a more idiomatic/modern JS style.</p>
<p>If this causes difficulties for your project, we are <a href="https://github.com/microsoft/typescript-go/issues">open to feedback on the issue tracker</a>.</p>
<h3 id="editor-support--lsp-progress">Editor Support &amp; LSP Progress</h3>
<p>When we released the Corsa codebase, it included a very rudimentary LSP-based language server.
While most tangible development has been on the compiler itself, we have been iterating on multiple fronts to port our editor functionality in this new system.
Because the Strada codebase communicates with editors via the TSServer format that predates LSP, we are not aiming to do a perfect 1:1 port between the codebases.
This means that porting code often requires more manual porting and has required a bit more up-front thought in how we generate type definitions that conform to LSP.
Gathering errors/diagnostics, go-to-definition, and hover work in very early stages.</p>
<p>Most recently, we have hit another milestone: we have enabled completions!
While auto-imports and other features around completions are not fully ported, this may be enough for many teams in large codebases.
Going forward, our priorities are in porting over our existing language server test suite, along with enabling find-all-references, rename, and signature help.</p>
<h3 id="api-progress">API Progress</h3>
<p>A big challenge as part of this port will be continuity with API consumers of TypeScript.
We have the initial foundation of an API layer that can be leveraged over standard I/O.
This work means that API consumers can communicate with a TypeScript process through IPC regardless of the consuming language.
Since we know that many API consumers will be writing TypeScript and JavaScript code, we also have JavaScript-based clients for interacting with the API.</p>
<p>Because so much TypeScript API usage today is synchronous, we wanted to make it possible to communicate with this process in a synchronous way.
Node.js unfortunately doesn&#8217;t provide an easy way to communicate synchronously with a child process, so we developed a native Node.js module in Rust (which should make lots of people happy) called <a href="https://github.com/microsoft/libsyncrpc">libsyncrpc</a>.</p>
<p>We are still in the early days of API design here, but we are open to thoughts and feedback on the matter.
More <a href="https://github.com/microsoft/typescript-go/pull/711">details about the current API server are available here</a>.</p>
<h2 id="known-and-notable-differences">Known and Notable Differences</h2>
<p>As we&#8217;ve mentioned, the Corsa compiler and language service may still have some differences from Strada.
There are some differences that you may hit early on in trying out <code>tsgo</code> and the Native Preview VS Code extension.</p>
<p>Some of those come from eventual <a href="https://github.com/microsoft/TypeScript/issues/54500">TypeScript 6.0 deprecations</a>, like <code>node</code>/<code>node10</code> resolution (in favor of <code>node16</code>, <code>nodenext</code>, and <code>bundler</code>).
If you use <code>--moduleResolution node</code> or <code>--module commonjs</code>, you may see some errors like:</p>
<pre class="language-text" style="padding: 10px; border-radius: 10px;"><code>Cannot find module 'blah' or its corresponding type declarations.
Module '"module"' has no exported member 'Thing'.
</code></pre>
<p>You will get consistent errors if you switch your <code>tsconfig.json</code> settings to use</p>
<pre class="prettyprint language-json" style="padding: 10px; border-radius: 10px;"><code>{
    "compilerOptions": {
        // ...
        "module": "preserve",
        "moduleResolution": "bundler",
    }
}
</code></pre>
<p>or</p>
<pre class="prettyprint language-json" style="padding: 10px; border-radius: 10px;"><code>{
    "compilerOptions": {
        // ...
        "module": "nodenext"
    }
}
</code></pre>
<p>These can be manually fixed depending on your configuration, though often you can remove the imports and leverage auto-imports to do &#8220;the right thing&#8221;.</p>
<p>Beyond deprecations, downlevel emit to older targets is limited, and JSX emit only works as far as preserving what you wrote.
Declaration emit is currently not supported either.
<code>--build</code> mode and language service functionality around project references is still not available, though project dependencies can be built through <code>tsc</code>, and the native preview language service can often leverage generated <code>.d.ts</code> files.</p>
<h2 id="whats-next">What&#8217;s Next?</h2>
<p>By later this year, we will aim to have a more complete version of our compiler with major features like <code>--build</code>, along with most language service features for editors.</p>
<p>But we don&#8217;t expect you to wait that long!
As TypeScript Native Previews are published nightly, we&#8217;ll aim to provide periodic updates on major notable developments.
So give the native previews a shot!</p>
<p>Happy Hacking!</p>
<p>&#8211; Daniel Rosenwasser and the TypeScript Team</p>
<p>The post <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-native-previews/">Announcing TypeScript Native Previews</a> appeared first on <a href="https://devblogs.microsoft.com/typescript">TypeScript</a>.</p>
