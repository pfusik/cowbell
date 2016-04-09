Using Cowbell
=============

A few things to know before we get started:

* Cowbell provides a collection of player backends to support different file types. It's your responsibility to select a player backend that can actually the file you want to play - Cowbell does not auto-detect this.
* Most player backends (in fact, all of them except `AudioPlayer`) use `XMLHttpRequest` to load the music files. This means that the files have to be hosted either at a local URL, or a web server that has been configured to return CORS HTTP headers.

Importing scripts
-----------------

Cowbell is distributed as a set of minified scripts (if you compiled Cowbell from source, you can find these in the `dist` folder), allowing you to load just the specific player backends you're interested in. At minimum, you need `cowbell.min.js`, which gives you the ability to play browser-native formats such as MP3; for other more interesting formats, you need to import the relevant add-on modules as indicated in 'Available backends' below. For example, to play .VTX files, you'll need something like this on your page:

    <script src="cowbell/cowbell.min.js"></script>
    <script src="cowbell/ay-chip.min.js"></script>
    <script src="cowbell/vtx.min.js"></script>

Simple usage
------------

For a simple player widget (where you just have one track to play, and don't want to load new tracks), you can use the `Cowbell.createPlayer(containerElement, options)` function:

    <script>
        Cowbell.createPlayer('player-container', {
            'url': 'music/ambpower.mod',
            'player': Cowbell.Player.OpenMPT,
            'ui': Cowbell.UI.Basic,
            'playerOpts': {
                'pathToLibOpenMPT': '../dist/cowbell/libopenmpt.js'
            },
            'trackOpts': {}
        });
    </script>
    
    <div id="player-container"></div>

The `containerElement` parameter is the ID of the element that the player UI should be inserted into, or the DOM element object itself.

All fields in `options` are optional, but you'll probably want to specify `'url'` and `'player'` in order for the player to do something useful.

* `url`: The URL of the file to be played
* `player`: The player backend module to use (see 'Available backends' below)
* `ui`: The user interface module to use. Currently `Cowbell.UI.Basic` is the only one available (and is the default)
* `playerOpts`: A set of configuration options specific to this player backend
* `trackOpts`: A set of configuration options specific to this track, such as stereo separation

jQuery integration
------------------

If you're using jQuery on your page, an alternative jQuery-friendly syntax is available:

    $('#player-container').cowbell({
        'url': 'music/ambpower.mod',
        'player': Cowbell.Player.OpenMPT
    });

Step-by-step usage
------------------

For greater control over the way the player is put together - particularly if you want to be able to play multiple tracks through a single player widget - you can construct and connect up the components yourself.

1. Import the relevant scripts as before:

        <script src="../dist/cowbell/cowbell.min.js"></script>
        <script src="../dist/cowbell/openmpt.min.js"></script>

2. Create a Player object of the appropriate type:

        var psgPlayer = new Cowbell.Player.PSG();

    Additional player config options can be passed as a parameter:

        var modPlayer = new Cowbell.Player.OpenMPT({
            'pathToLibOpenMPT': '../dist/cowbell/libopenmpt.js'
        });

3. Retrieve a Track object from the Player:

        var track = new modPlayer.Track('music/ambpower.mod');
        
    Additional track configuration options can be passed as a second parameter:
    
        var track = new stcPlayer.Track('music/worlds_apart.stc', {
            'stereoMode': 'acb';
        });

4. Create a UI object to display the player controls, passing the container DOM element as a parameter:

        var container = document.getElementById('player');
        var playerUI = new Cowbell.UI.Basic(container);
        
5. Open the track in the player UI:

        playerUI.open(track);
        
    `open` can be called again on the UI object at any time, to switch to a new track. Note that the audio file will only be downloaded at the point the user begins playback, not on the call to `open` - this means you can safely call `open` on page load without wasting bandwidth on a file that might never get played.

Available backends
------------------

### Cowbell.Player.Audio

*Imports needed:* `cowbell.min.js`

This provides support for the file types that your browser can play natively through the `<audio>` tag. On most browsers this includes MP3 and OGG, and possibly things like FLAC and WAV too.

*Options:* None

### Cowbell.Player.OpenMPT

*Imports needed:* `cowbell.min.js`, `openmpt.min.js`, `libopenmpt.js` (optional, see below)

A JavaScript port of the [libopenmpt](http://lib.openmpt.org/libopenmpt/) library, which provides highly accurate playback of a wide variety of PC / Amiga tracker formats. The full list of supported file types is:

mod s3m xm it mptm stm nst m15 stk wow ult 669 mtm med far mdl ams dsm amf okt dmf ptm psm mt2 dbm digi imf j2b gdm umx plm mo3 xpk ppm mmcmp


Since the main `libopenmpt.js` engine is over 2Mb in size, we recommend importing this 'lazily' via the `pathToLibOpenMPT` option, rather than importing it with a `<script>` tag as usual - that way, it will only be downloaded when the user starts playback, rather than on every page load.

*Options:*

* `pathToLibOpenMPT` - the URL to libopenmpt.js, if this has not been imported directly via a `<script>` tag

### Cowbell.Player.PSG

*Imports needed:* `cowbell.min.js`, `ay_chip.min.js`

Player for the .PSG format, an uncompressed stream of AY / YM sound chip command data.

*Options:* None

### Cowbell.Player.VTX

*Imports needed:* `cowbell.min.js`, `ay_chip.min.js`, `vtx.min.js`

Player for the .VTX format, an LH5-compressed stream of AY / YM sound chip command data.

*Options:* None

## Cowbell.Player.ZXSTC

*Imports needed:* `cowbell.min.js`, `ay_chip.min.js`, `zx.min.js`

Player for the .STC (ZX Spectrum Soundtracker) format, running the original player routine under Z80 emulation.

*Options: (can be specified as either player or track opts)*

* `ayFrequency`: Clock frequency (in Hz) of the AY chip. Default: 1773400
* `commandFrequency`: Frequency (in Hz) at which command data will be pushed to the AY chip. Default: 50
* `stereoMode`: A string indicating the stereo arrangement of the AY channels; for example, `'ACB'` denotes channel A on the left, B on the right, and C in the centre. Supported values are `'ABC'`, `'ACB'`, `'BAC'`, `'BCA'`, `'CAB'` and `'CBA'`; any other value denotes mono output. Default: mono
* `panning`: A three-element array indicating the exact stereo positioning of the AY channels, where 0.0 = hard left and 1.0 = hard right. For example, `[0.0, 1.0, 0.5]` denotes ACB stereo with full separation. If both `panning` and `stereoMode` are specified, `panning` takes precedence.


## Cowbell.Player.ZXPT3

*Imports needed:* `cowbell.min.js`, `ay_chip.min.js`, `zx.min.js`

Player for the .PT3 (ZX Spectrum ProTracker 3) format, running the universal Vortex Tracker 2 player routine under Z80 emulation.

*Options:* Same as for `Cowbell.Player.ZXSTC`