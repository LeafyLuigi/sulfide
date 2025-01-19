# Sulfide

This repo is for the "Sulfide" Discord Theme CSS class list.

The basis for the backend was created by [Zerthox](https://github.com/Zerthox) originally from his long-abandoned [DiscordSelectors](https://github.com/zerthox/discordselectors) repo. All functional code is written in [SCSS](https://sass-lang.com/), specifically targeting Dart Sass. [See Sass' documentation for installation](https://sass-lang.com/install/).

For support, feel free to join the [Pyrite Support Server](https://discord.gg/EeQQTWbTf5). A channel exists for updates to be forwarded to your server as well.

## Usage

### Prerequisites

You'll need `sass` installed using the instructions from the link above. This currently targets Node.js' Sass version 1.83.4 and above.

You should then be able to run `sass` in your command line, though the class list will likely work using another implementation.

Recommended `npm` functions:
```JSON
"buildTheme": "sass path/to/inputFiles:path/to/outputFiles -I path/to/loadPath",
"debugClassList": "sass path/to/_debug-dump-all.scss path/to/output.css -I path/to/loadPath"
```

### Adding as a submodule

You can add the main `_classes.scss` file to your theme by adding this repo as a submodule. You can add this repo as a submodule by running the following command:
```bash
git submodule add https://github.com/LeafyLuigi/sulfide classes
```
This command will add this repo as a submodule in the "classes" directory in the repo's root. [GitHub have their own blog post on submodules that may have more information](https://github.blog/open-source/git/working-with-submodules/).

#### Updating the submodule

As this repo could (and should) be updated fairly regularly, you'll need to run this command, preferably before any builds or tests to ensure the class list is up to date.
```bash
git submodule update --init --recursive --remote
```

### Using `_classes.scss`

You'll need to understand [Sass](https://sass-lang.com/documentation/syntax/) to use this. The class list is written in a JSON-like format. The class list supports the following:
- Aliases
- Both IDs and Classes (the starting "." is not listed in the list), including combining classes.
- Dirty `@warn` call for locating class usage by prepending `!` to the class list class name
- Nesting maps for finer class precision
- :is() and :where() support

`#{class()}` does the all the heavy lifting and has two shorthand functions pointing to it, `#{c()}` and `#{C()}`. An example on calling the function is below. You should only ever need to call `#{class()}`, `#{c()}` or `#{C()}` within your SCSS.

Input (when file is correctly linked up)
```scss
:is(#{c(main dark)}, #{c(main light)}) #{c(main appMount)} {
	background: transparent;
}
```

`#{c(main dark)}` => This works by checking the class list under "main" to find "dark", which happens to be `theme-dark`. As no `!` is prepended to the class nor a `#`, a `.` is then prepended and it becomes `.theme-dark`. This repeats for each class called.

Expected output, substituting `appMount_suffix` for whatever the class is at present (currently `appMount_ea7e65`). 
```css
:is(.theme-dark, .theme-light) .appMount_suffix {
	background: transparent;
}
```

The way I use in my theme, [Pyrite](https://github.com/LeafyLuigi/discord-themes/tree/master/pyrite), is extremely hacky. Using a Symlink at `$pyriteRoot/classes/_classes.scss` pointing to the SCSS file can be done but isn't recommended. Having a "backend" directory with `@forward` pointing to the class list then calling `@use "backend" as *;` in all `.scss` should work.

<details><summary>File examples</summary>

`source/base.scss` | Main "root" file that Pyrite uses.
```scss
@use "backend" as *;
@use "theme"; // Directory with all SCSS inside.
```

`source/backend/_index.scss`
```scss
@forward "default-variables";
@forward "mixins";
@forward "classes"; // Classes reside here, but it doesn't exist in source/backend/_classes.scss
```

`source/classes/_index.scss` (included when creating this repo as a submodule)
```scss
@forward "classes"; // Where "_classes.scss" resides.
```

`source/theme/_index.scss` File that points to other directories that contains more SCSS. There's more than just one directory in the actual file.
```scss
@forward "friends"; // We'll use "friends" as an example.
```

`source/theme/friends/_index.scss`
```scss
@forward "friends"; // Name of the file containing SCSS
@forward "messageRequests"; // Multiple files can be listed.
```

`source/theme/friends/_friends.scss`
```scss
@use "backend" as *; // THIS MUST BE CALLED AT THE START OF ALL SCSS FILES CONTAINING USED FUNCTIONS.
/* Start Friends Area */
:is(#{c(main dark)},#{c(main light)}) {
	:is(#{c(friends container)},#{c(friends multipleIconWrapper)}) {
		background: transparent;
	}
}
```

</details>

<details><summary>A small snippet from the "test" section from `_classes.scss` is below:</summary>

```scss
$classes: (
	"test": (
		// category to test simple shit; foo = bar, foo2 = bar2
		"a": (
			"foo": "bar",
			"foo2": "bar2",
		),
		// category to test different base name, "a foo" != "b foo". bar2 is example in this case.
		"b": (
			"foo": "bar2",
		),
		// category to test for quick debug traces
		"c": (
			"a": "!a",
			"b": "!b",
			"foo": "!bar",
		),
		// category for alias
		"d": (
			"foobar": "alias test a foo", // output should be "bar"
			"loopback": "alias test d loopback" // should error; this is usually commented in the _classes.scss file
		),
		// category to test dots (for class names) and hashes (for ids) etc
		"e": (
			"dot": ".dot", // should throw error, this list does not need "." before classes
			"hash": "#hash", // should not throw error, generation explicitly checks for "#" at start
			"multi": "class1.class2", // should not throw error
		),
		// category to test lists outputting to :is(<contents of list>)
		"f": (
			"foo": ("item1" "item2" "item3"), // outputs a :where() list
			"spaceSep": "item4" "item5" "item6", // outputs an :is() list
			"brackets": ["item7" "item8" "item9" "item with spaces"], // also outputs a :where() list
		),
	),
);
```

</details>

<details>
<summary>Calling each in order as `#{class(test $section $item)} {}` (ie `#{class(test a foo)} {} #{class(test a bar)} {} #{class(test b foo)} {} ...`) should output the following, excluding the comments:</summary>

```css
/* test a foo */ .bar {}
/* test a foo2 */ .bar2 {}

/* test b foo */ .bar2 {}

/* test c a */ .a {} /* Console should have a trace back to where `#{class(test c a)} was used. */
/* test c b */ .b {} /* Same as above */
/* test c foo */ .bar {} /* Same as above */

/* test d foobar */ .bar {} /* Uses the first "bar" in the list. If that changes, this changes. */
/* test d loopback */ /* "alias test d loopback" would fail as pointing an alias to another alias is disallowed */

/* test e dot */ /* ".dot" throws an error as the #{class()} function prepends a "." by default, throwing an error if a "." is found. */
/* test e hash */ #hash {} /* "#" is a special case to allow for IDs. */
/* test e multi */ .class1.class2 {} /* Though allowed, not entirely recommended for normal usage. */

/* test f foo */ :where(.item1,.item2,.item3) {}
/* test f spaceSep */ :is(.item4,.item5,.item6) {}
/* test f brackets */ :where(.item7,.item8,.item9,.item with spaces) {} /* ".item with spaces" is invalid CSS but is still outputted. */
```

</details>

Run the project's `buildTheme` function (or equivalent) to see the final output.

### Debugging/Dumping `_classes.scss`

Included in this repo is `_debug-dump-all.scss` which is a file that contains two main functions, `debug_dump_all_classes_yes_i_am_sure()` and `debug_dump_all_aliases_yes_i_am_sure()`. Uncomment (by removing the desired function's prepended `//`) then run your project's `debugClassList` function.

The former will output a parsed class list containing every CSS class (and ID), excluding aliases. This way a whole list containing (hopefully) unique classes can be dealt with on its own, such as for finding out-of-date classes or finding duplicates.

The latter will output a parsed list containing all CSS classes (and IDs) with an alias linked to it. This will not output the aliases used to point to the class/ID though.

## Want to contribute?

Feel free to open up a pull request with your edits to `_classes.scss`. Any new classes should go into a category relevant to the part of the app the CSS class is found. If you're moving existing classes, replacing the old class with an alias pointing to the new class is required. Removing an existing class (or alias) should not be done without approval.