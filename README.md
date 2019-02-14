# scratch3_eim_mpfshell

How to add to scratch3 ?

Have the following requirements base on [codelab_adapter_extensions](https://github.com/Scratch3Lab/codelab_adapter_extensions).

1. [scratch-gui](https://github.com/LLK/scratch-gui)
2. [scratch-vm](https://github.com/LLK/scratch-vm)
3. [scratch3_eim](https://github.com/Scratch3Lab/scratch3_eim)

## scratch-gui

move `scratch3_mpfshell` to `\scratch-vm\src\extensions\scratch3_mpfshell`.

open `index.jsx` in `\Scratch3\scratch-gui\src\lib\libraries\extensions` .

``` javascript
export default [
    // After the other plug-ins
	,{
        name: 'Mpfshell',
        extensionId: 'mpfshell',
        // iconURL: mpfshellImage, // add icon `import mpfshellImage from './mpfshell.png';` if your need.
        description: (
            <FormattedMessage
                defaultMessage="Mpfshell for Scratch3"
                description="Use mpfshell control micropython."
                id="gui.extension.Mpfshell.description"
            />
        ),
        featured: true
    }
]
```

![](readme/gui.png)

However, it does not work yet. You need to provide this plugin in your scratch-vm.

## scratch-vm

open `extension-manager.js` in `\Scratch3\scratch-vm\src\extension-support` .

```javascript
const Scratch3EimBlocks = require('../extensions/scratch3_eim');
const Scratch3MpfshellBlocks = require('../extensions/scratch3_mpfshell');

const builtinExtensions = {
    // After the other Extensions
    eim: Scratch3EimBlocks, // add eim extension
    mpfshell: Scratch3MpfshellBlocks, // add mpfshell extension
};
```

This completes the plug-in offering.

If you click on the plugin and nothing happens, you need to re-click `yarn unlink` and `yarn link` in `scratch-vm`.

## scratch-mpfshell

click image directory.

![](readme/codelab_adapter.png)

move `extensions` to `codelab_adapter\extensions`.

![](readme/checked.png)

checked extension_mpfshell start mpfshell.

into scratch3 select mpfshell extension, let's play!

![](readme/demo.png)

### add bpi:bit support!

We can use micropython firmware based on mpfshell.

According to the building blocks of mpfshell, we can make control blocks of other hardware. Here I take bpibit as an example.

![](readme/bpibit.png)

So we can convert the above definition into a built-in building block.

see `scratch3_bpibit/index.js`

Take this as an example(same as mpfshell).

1. in `extension-manager.js`

```javascript
const Scratch3BpibitBlocks = require('../extensions/scratch3_bpibit');

const builtinExtensions = {
    bpibit: Scratch3BpibitBlocks
};
```

2. in `index.jsx`

```
export default [
    // After the other plug-ins
    ,{
        name: 'Bpibit',
        extensionId: 'bpibit',
        collaborator: 'Bananapi',
        description: (
            <FormattedMessage
                defaultMessage="Bpibit for Scratch3"
                description="bpibit"
                id="gui.extension.bpibit.description"
            />
        ),
        featured: true
    }
]
```

3. in `index.js`

```javascript
const ArgumentType = require('../../extension-support/argument-type');
const BlockType = require('../../extension-support/block-type');
const formatMessage = require('format-message');
// const MathUtil = require('../../util/math-util');

const Scratch3MpfshellBlocks = require('../scratch3_mpfshell');

class Scratch3BpibitBlocks {
    constructor (runtime) {
        /**
         * The runtime instantiating this block package.
         * @type {Runtime}
         */
        this.runtime = runtime;
        this.mpfshell = new Scratch3MpfshellBlocks();
    }

    /**
     * The key to load & store a target's pen-related state.
     * @type {string}
     */
    static get STATE_KEY () {
        return 'Scratch.bpibit';
    }

    /**
     * @returns {object} metadata for this extension and its blocks.
     */
    getInfo () {
        return {
            id: 'bpibit',
            name: formatMessage({
                id: 'bpibit.categoryName',
                default: 'Bpibit',
                description: 'Use it to control your micropython'
            }),
            // menuIconURI: menuIconURI,
            blockIconURI: blockIconURI,
            // showStatusButton: true,
            blocks: [
                {
                    opcode: 'isconnected',
                    blockType: BlockType.REPORTER,
                    arguments: {}
                },
                {
                    opcode: 'connect',
                    blockType: BlockType.COMMAND,
                    text: formatMessage({
                        id: 'bpibit.connect',
                        default: 'connect [DATA]',
                        description: 'connect micropython device.'
                    }),
                    arguments: {
                        DATA: {
                            type: ArgumentType.STRING,
                            defaultValue: formatMessage({
                                id: 'bpibit.defaultArgsToOpen',
                                default: '',
                                description: 'connect device name(default is none to find).'
                            })
                        }
                    }
                },
                {
                    opcode: 'display',
                    blockType: BlockType.COMMAND,
                    text: formatMessage({
                        id: 'bpibit.dispaly',
                        default: 'dispaly [DATA]',
                        description: 'dispaly led pixel'
                    }),
                    arguments: {
                        DATA: {
                            type: ArgumentType.STRING,
                            defaultValue: formatMessage({
                                id: 'bpibit.defaultTextToDispaly',
                                default: 'hello world!',
                                description: 'bpibit display.'
                            })
                        }
                    }
                }
            ],
            menus: {

            }
        };
    }

    display (args) {
        const cmd = "display.scroll('" + args.DATA + "')";
        // console.log(cmd);
        this.mpfshell.exec({mutation: null, TOPIC: 'eim/mpfshell/exec', DATA: cmd});
    }

    connect (DATA) {
        this.mpfshell.open(DATA);
        this.mpfshell.exec({mutation: null, TOPIC: 'eim/mpfshell/exec', DATA: 'from microbit import *'});
    }

    isconnected () {
        this.mpfshell.isconnected();

        return this.mpfshell.return_info();
    }

}

module.exports = Scratch3BpibitBlocks;

```

then your see this.

![](readme/bit.png)

![](readme/list.png)

make hardware for scratch3 by yourself. let go!