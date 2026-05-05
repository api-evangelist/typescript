---
title: "Announcing TypeScript 5.9"
url: "https://devblogs.microsoft.com/typescript/announcing-typescript-5-9/"
date: "Fri, 01 Aug 2025 16:19:25 +0000"
author: "Daniel Rosenwasser"
feed_url: "https://devblogs.microsoft.com/typescript/feed/"
---
<p>Today we are excited to announce the release of TypeScript 5.9!</p>
<p>If you&#8217;re not familiar with TypeScript, it&#8217;s a language that builds on JavaScript by adding syntax for types.
With types, TypeScript makes it possible to check your code to avoid bugs ahead of time.
The TypeScript type-checker does all this, and is also the foundation of great tooling in your editor and elsewhere, making coding even easier.
If you&#8217;ve written JavaScript in editors like Visual Studio and VS Code, TypeScript even powers features you might already be using like completions, go-to-definition, and more.
You can <a href="https://typescriptlang.org/">learn more about TypeScript at our website</a>.</p>
<p>But if you&#8217;re already familiar, you can start using TypeScript 5.9 today!</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npm install -D typescript
</code></pre>
<p>Let&#8217;s take a look at what&#8217;s new in TypeScript 5.9!</p>
<ul>
<li><a href="#minimal-and-updated-tsc---init">Minimal and Updated <code>tsc --init</code></a></li>
<li><a href="#support-for-import-defer">Support for <code>import defer</code></a></li>
<li><a href="#support-for---module-node20">Support for <code>--module node20</code></a></li>
<li><a href="#summary-descriptions-in-dom-apis">Summary Descriptions in DOM APIs</a></li>
<li><a href="#expandable-hovers-preview">Expandable Hovers (Preview)</a></li>
<li><a href="#configurable-maximum-hover-length">Configurable Maximum Hover Length</a></li>
<li><a href="#optimizations">Optimizations</a></li>
<li><a href="#notable-behavioral-changes">Notable Behavioral Changes</a></li>
</ul>
<h2 id="whats-new-since-the-beta-and-rc">What&#8217;s New Since the Beta and RC?</h2>
<p>There have been no changes to TypeScript 5.9 since the release candidate.</p>
<p>A few fixes for reported issues have been made <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-5-9-beta/">since the 5.9 beta</a>, including the restoration of <code>AbortSignal.abort()</code> to the DOM library.
Additionally, we have added a section about <a href="#notable-behavioral-changes">Notable Behavioral Changes</a>.</p>
<h2 id="minimal-and-updated-tsc---init">Minimal and Updated <code>tsc --init</code></h2>
<p>For a while, the TypeScript compiler has supported an <code>--init</code> flag that can create a <code>tsconfig.json</code> within the current directory.
In the last few years, running <code>tsc --init</code> created a very &#8220;full&#8221; <code>tsconfig.json</code>, filled with commented-out settings and their descriptions.
We designed this with the intent of making options discoverable and easy to toggle.</p>
<p>However, given external feedback (and our own experience), we found it&#8217;s common to immediately delete most of the contents of these new <code>tsconfig.json</code> files.
When users want to discover new options, we find they rely on auto-complete from their editor, or navigate to <a href="https://www.typescriptlang.org/tsconfig/">the tsconfig reference on our website</a> (which the generated <code>tsconfig.json</code> links to!).
What each setting does is also documented on that same page, and can be seen via editor hovers/tooltips/quick info.
While surfacing some commented-out settings might be helpful, the generated <code>tsconfig.json</code> was often considered overkill.</p>
<p>We also felt that it was time that <code>tsc --init</code> initialized with a few more prescriptive settings than we already enable.
We looked at some common pain points and papercuts users have when they create a new TypeScript project.
For example, most users write in modules (not global scripts), and <code>--moduleDetection</code> can force TypeScript to treat every implementation file as a module.
Developers also often want to use the latest ECMAScript features directly in their runtime, so <code>--target</code> can typically be set to <code>esnext</code>.
JSX users often find that going back to set <code>--jsx</code> is needless friction, and its options are slightly confusing.
And often, projects end up loading more declaration files from <code>node_modules/@types</code> than TypeScript actually needs; but specifying an empty <code>types</code> array can help limit this.</p>
<p>In TypeScript 5.9, a plain <code>tsc --init</code> with no other flags will generate the following <code>tsconfig.json</code>:</p>
<pre class="prettyprint language-json" style="padding: 10px; border-radius: 10px;"><code>{
  // Visit https://aka.ms/tsconfig to read more about this file
  "compilerOptions": {
    // File Layout
    // "rootDir": "./src",
    // "outDir": "./dist",

    // Environment Settings
    // See also https://aka.ms/tsconfig_modules
    "module": "nodenext",
    "target": "esnext",
    "types": [],
    // For nodejs:
    // "lib": ["esnext"],
    // "types": ["node"],
    // and npm install -D @types/node

    // Other Outputs
    "sourceMap": true,
    "declaration": true,
    "declarationMap": true,

    // Stricter Typechecking Options
    "noUncheckedIndexedAccess": true,
    "exactOptionalPropertyTypes": true,

    // Style Options
    // "noImplicitReturns": true,
    // "noImplicitOverride": true,
    // "noUnusedLocals": true,
    // "noUnusedParameters": true,
    // "noFallthroughCasesInSwitch": true,
    // "noPropertyAccessFromIndexSignature": true,

    // Recommended Options
    "strict": true,
    "jsx": "react-jsx",
    "verbatimModuleSyntax": true,
    "isolatedModules": true,
    "noUncheckedSideEffectImports": true,
    "moduleDetection": "force",
    "skipLibCheck": true,
  }
}
</code></pre>
<p>For more details, see the <a href="https://github.com/microsoft/TypeScript/pull/61813">implementing pull request</a> and <a href="https://github.com/microsoft/TypeScript/issues/58420">discussion issue</a>.</p>
<h2 id="support-for-import-defer">Support for <code>import defer</code></h2>
<p>TypeScript 5.9 introduces support for <a href="https://github.com/tc39/proposal-defer-import-eval/">ECMAScript&#8217;s deferred module evaluation proposal</a> using the new <code>import defer</code> syntax.
This feature allows you to import a module without immediately executing the module and its dependencies, providing better control over when work and side-effects occur.</p>
<p>The syntax only permits namespace imports:</p>
<pre class="prettyprint language-typescript" style="padding: 10px; border-radius: 10px;"><code>import defer * as feature from "./some-feature.js";
</code></pre>
<p>The key benefit of <code>import defer</code> is that the module is only evaluated when one of its exports is first accessed.
Consider this example:</p>
<pre class="prettyprint language-typescript" style="padding: 10px; border-radius: 10px;"><code>// ./some-feature.ts
initializationWithSideEffects();

function initializationWithSideEffects() {
  // ...
  specialConstant = 42;

  console.log("Side effects have occurred!");
}

export let specialConstant: number;
</code></pre>
<p>When using <code>import defer</code>, the <code>initializationWithSideEffects()</code> function will not be called until you actually access a property of the imported namespace:</p>
<pre class="prettyprint language-typescript" style="padding: 10px; border-radius: 10px;"><code>import defer * as feature from "./some-feature.js";

// No side effects have occurred yet

// ...

// As soon as `specialConstant` is accessed, the contents of the `feature`
// module are run and side effects have taken place.
console.log(feature.specialConstant); // 42
</code></pre>
<p>Because evaluation of the module is deferred until you access a member off of the module, you cannot use named imports or default imports with <code>import defer</code>:</p>
<pre class="prettyprint language-typescript" style="padding: 10px; border-radius: 10px;"><code>// &#x274c; Not allowed
import defer { doSomething } from "some-module";

// &#x274c; Not allowed  
import defer defaultExport from "some-module";

// &#x2705; Only this syntax is supported
import defer * as feature from "some-module";
</code></pre>
<p>Note that when you write <code>import defer</code>, the module and its dependencies are fully loaded and ready for execution.
That means that the module will need to exist, and will be loaded from the file system or a network resource.
The key difference between a regular <code>import</code> and <code>import defer</code> is that <em>the execution of statements and declarations</em> is deferred until you access a property of the imported namespace.</p>
<p>This feature is particularly useful for conditionally loading modules with expensive or platform-specific initialization. It can also improve startup performance by deferring module evaluation for app features until they are actually needed.</p>
<p>Note that <code>import defer</code> is not transformed or &#8220;downleveled&#8221; at all by TypeScript.
It is intended to be used in runtimes that support the feature natively, or by tools such as bundlers that can apply the appropriate transformation.
That means that <code>import defer</code> will only work under the <code>--module</code> modes <code>preserve</code> and <code>esnext</code>.</p>
<p>We&#8217;d like to extend our thanks to <a href="https://github.com/nicolo-ribaudo">Nicolò Ribaudo</a> who championed the proposal in TC39 and also provided <a href="https://github.com/microsoft/TypeScript/pull/60757">the implementation for this feature</a>.</p>
<h2 id="support-for---module-node20">Support for <code>--module node20</code></h2>
<p>TypeScript provides several <code>node*</code> options for the <code>--module</code> and <code>--moduleResolution</code> settings.
Most recently, <code>--module nodenext</code> has supported the ability to <code>require()</code> ECMAScript modules from CommonJS modules, and correctly rejects import assertions (in favor of the standards-bound <a href="https://github.com/tc39/proposal-import-attributes">import attributes</a>).</p>
<p>TypeScript 5.9 brings a stable option for these settings called <code>node20</code>, intended to model the behavior of Node.js v20.
This option is unlikely to have new behaviors in the future, unlike <code>--module nodenext</code> or <code>--moduleResolution nodenext</code>.
Also unlike <code>nodenext</code>, specifying <code>--module node20</code> will imply <code>--target es2023</code> unless otherwise configured.
<code>--module nodenext</code>, on the other hand, implies the floating <code>--target esnext</code>.</p>
<p>For more information, <a href="https://github.com/microsoft/TypeScript/pull/61805">take a look at the implementation here</a>.</p>
<h2 id="summary-descriptions-in-dom-apis">Summary Descriptions in DOM APIs</h2>
<p>Previously, many of the DOM APIs in TypeScript only linked to the MDN documentation for the API.
These links were useful, but they didn&#8217;t provide a quick summary of what the API does.
Thanks to a few changes from <a href="https://github.com/Bashamega">Adam Naji</a>, TypeScript now includes summary descriptions for many DOM APIs based on the MDN documentation.
You can see more of these changes <a href="https://github.com/microsoft/TypeScript-DOM-lib-generator/pull/1993">here</a> and <a href="https://github.com/microsoft/TypeScript-DOM-lib-generator/pull/1940">here</a>.</p>
<h2 id="expandable-hovers-preview">Expandable Hovers (Preview)</h2>
<p><em>Quick Info</em> (also called &#8220;editor tooltips&#8221; and &#8220;hovers&#8221;) can be very useful for peeking at variables to see their types, or at type aliases to see what they actually refer to.
Still, it&#8217;s common for people to want to <em>go deeper</em> and get details from whatever&#8217;s displayed within the quick info tooltip.
For example, if we hover our mouse over the parameter <code>options</code> in the following example:</p>
<pre class="prettyprint language-typescript" style="padding: 10px; border-radius: 10px;"><code>export function drawButton(options: Options): void
</code></pre>
<p>We&#8217;re left with <code>(parameter) options: Options</code>.</p>
<p><img alt="Tooltip for a parameter declared as which just shows ." src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/06/bare-hover-5.8-01.png" /></p>
<p>Do we really need to jump to the definition of the type <code>Options</code> just to see what members this value has?</p>
<p>Previously, that was actually the case.
To help here, TypeScript 5.9 is now previewing a feature called <em>expandable hovers</em>, or &#8220;quick info verbosity&#8221;.
If you use an editor like VS Code, you&#8217;ll now see a <code>+</code> and <code>-</code> button on the left of these hover tooltips.
Clicking on the <code>+</code> button will expand out types more deeply, while clicking on the <code>-</code> button will collapse to the last view.</p>
<p><video height="150" loop="loop" src="https://devblogs.microsoft.com/typescript/wp-content/uploads/sites/11/2025/06/expandable-quick-info-1.mp4" style="width: 100%;" width="300"></video></p>
<p>This feature is currently in preview, and we are seeking feedback for both TypeScript and our partners on Visual Studio Code.
For more details, see <a href="https://github.com/microsoft/TypeScript/pull/59940">the PR for this feature here</a>.</p>
<h2 id="configurable-maximum-hover-length">Configurable Maximum Hover Length</h2>
<p>Occasionally, quick info tooltips can become so long that TypeScript will truncate them to make them more readable.
The downside here is that often the most important information will be omitted from the hover tooltip, which can be frustrating.
To help with this, TypeScript 5.9&#8217;s language server supports a configurable hover length, which can be configured in VS Code via the <code>js/ts.hover.maximumLength</code> setting.</p>
<p>Additionally, the new default hover length is substantially larger than the previous default.
This means that in TypeScript 5.9, you should see more information in your hover tooltips by default.
For more details, see <a href="https://github.com/microsoft/TypeScript/pull/61662">the PR for this feature here</a> and <a href="https://github.com/microsoft/vscode/pull/248181">the corresponding change to Visual Studio Code here</a>.</p>
<h2 id="optimizations">Optimizations</h2>
<h3 id="cache-instantiations-on-mappers">Cache Instantiations on Mappers</h3>
<p>When TypeScript replaces type parameters with specific type arguments, it can end up instantiating many of the same intermediate types over and over again.
In complex libraries like Zod and tRPC, this could lead to both performance issues and errors reported around excessive type instantiation depth.
Thanks to <a href="https://github.com/microsoft/TypeScript/pull/61505">a change</a> from <a href="https://github.com/Andarist">Mateusz Burzyński</a>, TypeScript 5.9 is able to cache many intermediate instantiations when work has already begun on a specific type instantiation.
This in turn avoids lots of unnecessary work and allocations.</p>
<h3 id="avoiding-closure-creation-in-fileordirectoryexistsusingsource">Avoiding Closure Creation in <code>fileOrDirectoryExistsUsingSource</code></h3>
<p>In JavaScript, a function expression will typically allocate a new function object, even if the wrapper function is just passing through arguments to another function with no captured variables.
In code paths around file existence checks, <a href="https://github.com/VincentBailly">Vincent Bailly</a> found examples of these pass-through function calls, even though the underlying functions only took single arguments.
Given the number of existence checks that could take place in larger projects, he cited a speed-up of around 11%.
<a href="https://github.com/microsoft/TypeScript/pull/61822/">See more on this change here</a>.</p>
<h2 id="notable-behavioral-changes">Notable Behavioral Changes</h2>
<h3 id="libdts-changes"><code>lib.d.ts</code> Changes</h3>
<p>Types generated for the DOM may have an impact on type-checking your codebase.</p>
<p>Additionally, one notable change is that <code>ArrayBuffer</code> has been changed in such a way that it is no longer a supertype of several different <code>TypedArray</code> types.
This also includes subtypes of <code>UInt8Array</code>, such as <code>Buffer</code> from Node.js.
As a result, you&#8217;ll see new error messages such as:</p>
<pre class="language-text" style="padding: 10px; border-radius: 10px;"><code>error TS2345: Argument of type 'ArrayBufferLike' is not assignable to parameter of type 'BufferSource'.
error TS2322: Type 'ArrayBufferLike' is not assignable to type 'ArrayBuffer'.
error TS2322: Type 'Buffer' is not assignable to type 'Uint8Array&lt;ArrayBufferLike&gt;'.
error TS2322: Type 'Buffer' is not assignable to type 'ArrayBuffer'.
error TS2345: Argument of type 'Buffer' is not assignable to parameter of type 'string | Uint8Array&lt;ArrayBufferLike&gt;'.
</code></pre>
<p>If you encounter issues with <code>Buffer</code>, you may first want to check that you are using the latest version of the <code>@types/node</code> package.
This might include running</p>
<pre class="language-text" style="padding: 10px; border-radius: 10px;"><code>npm update @types/node --save-dev
</code></pre>
<p>Much of the time, the solution is to specify a more specific underlying buffer type instead of using the default <code>ArrayBufferLike</code> (i.e. explicitly writing out <code>Uint8Array&lt;ArrayBuffer&gt;</code> rather than a plain <code>Uint8Array</code>).
In instances where some <code>TypedArray</code> (like <code>Uint8Array</code>) is passed to a function expecting an <code>ArrayBuffer</code> or <code>SharedArrayBuffer</code>, you can also try accessing the <code>buffer</code> property of that <code>TypedArray</code> like in the following example:</p>
<pre class="prettyprint language-diff" style="padding: 10px; border-radius: 10px;"><code>  let data = new Uint8Array([0, 1, 2, 3, 4]);
- someFunc(data)
+ someFunc(data.buffer)
</code></pre>
<h2 id="type-argument-inference-changes">Type Argument Inference Changes</h2>
<p>In an effort to fix &#8220;leaks&#8221; of type variables during inference, TypeScript 5.9 may introduce changes in types and possibly new errors in some codebases.
These are hard to predict, but can often be fixed by adding type arguments to generic functions calls.
<a href="https://github.com/microsoft/TypeScript/pull/61668">See more details here</a>.</p>
<h2 id="whats-next">What&#8217;s Next?</h2>
<p>Now that TypeScript 5.9 is out, you might be wondering what&#8217;s in store for the next version: TypeScript 6.0.</p>
<p>As you might have heard, much of our recent focus has been on <a href="https://devblogs.microsoft.com/typescript/typescript-native-port/">the native port of TypeScript</a> which will eventually be available as TypeScript 7.0.
So what does that mean for TypeScript 6.0?</p>
<p>Our vision for TypeScript 6.0 is to act as a transition point for developers to adjust their codebases for TypeScript 7.0.
While TypeScript 6.0 may still ship updates and features, most users should think of it as a readiness check for adopting TypeScript 7.0.
This new version is meant to align with TypeScript 7.0, introducing deprecations around certain settings and possibly updating type-checking behavior in small ways.
Luckily, we don&#8217;t predict most projects will have too much trouble upgrading to TypeScript 6.0, and it will likely be entirely API compatible with TypeScript 5.9.</p>
<p>We&#8217;ll have more details coming soon.
That includes details on TypeScript 7.0 as well, where you can <a href="https://marketplace.visualstudio.com/items?itemName=TypeScriptTeam.native-preview">try it out in Visual Studio Code today</a> and <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-native-previews/">install it right in your project</a>.</p>
<p>Otherwise, we hope that TypeScript 5.9 treats you well, and makes your day-to-day coding a joy.</p>
<p>Happy Hacking!</p>
<p>&#8211; Daniel Rosenwasser and the TypeScript Team</p>
<p>The post <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-5-9/">Announcing TypeScript 5.9</a> appeared first on <a href="https://devblogs.microsoft.com/typescript">TypeScript</a>.</p>
