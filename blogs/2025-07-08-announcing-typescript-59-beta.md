---
title: "Announcing TypeScript 5.9 Beta"
url: "https://devblogs.microsoft.com/typescript/announcing-typescript-5-9-beta/"
date: "Tue, 08 Jul 2025 17:38:54 +0000"
author: "Daniel Rosenwasser"
feed_url: "https://devblogs.microsoft.com/typescript/feed/"
---
<p>Today we are excited to announce the availability of TypeScript 5.9 Beta.</p>
<p>To get started using the beta, you can get it through npm with the following command:</p>
<pre class="prettyprint language-sh" style="padding: 10px; border-radius: 10px;"><code>npm install -D typescript@beta
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
</ul>
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
JSX users often find that going back to set <code>--jsx</code> is a needless friction, and its options are slightly confusing.
And often, projects end up loading more declaration files from <code>node_modules/@types</code> than TypeScript actually needs &#8211; but specifying an empty <code>types</code> array can help limit this.</p>
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
<p>The key benefit of <code>import defer</code> is that the module is only evaluated on the first use. Consider this example:</p>
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
<p>To help here, TypeScript 5.9 is now previewing a feature called <em>expandable hovers</em>, or &#8220;quick info verbosity&#8221;.
If you use an editor like VS Code, you&#8217;ll now see a <code>+</code> and <code>-</code> button on the left of these hover tooltips.
Clicking on the <code>+</code> button will expand out types more deeply, while clicking on the <code>-</code> will go back to the last view.</p>
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
<h2 id="whats-next">What&#8217;s Next?</h2>
<p>As you might have heard, much of our recent focus has been on <a href="https://devblogs.microsoft.com/typescript/typescript-native-port/">the native port of TypeScript</a> which will eventually be available as TypeScript 7.
You can actually try out the native port today by checking out <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-native-previews/">TypeScript native previews</a>, which are released nightly.</p>
<p>However, we are still developing TypeScript 5.9 and addressing issues, and encourage you to try it out and give us feedback.
If you need a snapshot of TypeScript that&#8217;s newer than the beta, <a href="https://www.typescriptlang.org/docs/handbook/nightly-builds.html">TypeScript also has nightly releases</a>, and you can try out the latest editing experience by installing the <a href="https://marketplace.visualstudio.com/items?itemName=ms-vscode.vscode-typescript-next">JavaScript and TypeScript Nightly extension for Visual Studio Code</a>.</p>
<p>So we hope you try out the beta or a nightly release today and let us know what you think!</p>
<p>Happy Hacking!</p>
<p>&#8211; Daniel Rosenwasser and the TypeScript Team</p>
<p>The post <a href="https://devblogs.microsoft.com/typescript/announcing-typescript-5-9-beta/">Announcing TypeScript 5.9 Beta</a> appeared first on <a href="https://devblogs.microsoft.com/typescript">TypeScript</a>.</p>
