# RFID Cartridge Launcher

Drop a cartridge into the slot and its game starts. Take out a music cartridge and the song pauses; put it back and it carries on. Each cartridge hides an NFC sticker, read by a Raspberry Pi Pico and an RC522 reader, and a Python launcher on a Windows PC does the rest.

Built by [PiShop](https://www.pishop.co.za) for the rAge 2026 expo. It follows on from our earlier project, [From Card To Console: an RFID-Powered Game Launcher](https://blog.pishop.co.za/from-card-to-console-an-rfid-powered-game-launcher/).

![The visitor screen with a game cartridge lifted out of the shelf](docs/visitor-screen.png)

## How it works

1. An NFC sticker inside the cartridge sits over the RC522 reader.
2. The Pico reads the sticker's ID and sends it to the PC over USB.
3. `rfid_launcher.py` looks the ID up in `tags.json`.
4. Games start through Steam, a launch link or their program file. Music plays in VLC.
5. The visitor screen in the browser slides its shelf to that cartridge and lifts it out.

When the cartridge is removed, the Pico reports that too, which is how music pauses and resumes.

## Repository layout

```
pico/
  main.py              Reader program (runs on the Pico at power-up)
  mfrc522.py           RC522 driver for MicroPython
  rfid_test.py         Diagnostic: finds the best receiver gain for your reader
launcher/
  rfid_launcher.py     PC launcher, web server, visitor screen and setup page
  tags.example.json    Example settings and cartridges
  frontend/
    index.html         Visitor screen (cartridge shelf)
    manage.html        Cartridge setup page
    fonts/             Silkscreen and Chakra Petch (SIL Open Font License)
docs/                  Screenshots
```

## What you need

| Part | Notes |
| --- | --- |
| Raspberry Pi Pico | Pico, Pico W or Pico 2 |
| RC522 RFID reader | 13.56 MHz. Solder its header pins |
| NTAG213 NFC stickers | One per cartridge. MIFARE cards and fobs also work |
| Jumper wires and a micro-USB cable | |
| Windows PC | With [Python 3](https://www.python.org/downloads/) and [VLC](https://www.videolan.org/vlc/) |

## 1. Wire the reader

| RC522 | Pico pin | Physical pin |
| --- | --- | --- |
| 3.3V | 3V3(OUT) | 36 |
| GND | GND | 38 |
| SDA | GP17 | 22 |
| SCK | GP18 | 24 |
| MOSI | GP19 | 25 |
| MISO | GP16 | 21 |
| RST | GP20 | 26 |

Leave IRQ unconnected. Power the reader from 3.3V only, never 5V.

## 2. Set up the Pico

1. Hold BOOTSEL while plugging in the Pico, then drag the MicroPython `.uf2` for your board from [micropython.org/download](https://micropython.org/download/) onto the drive that appears.
2. In [Thonny](https://thonny.org), select **MicroPython (Raspberry Pi Pico)**.
3. Save `pico/mfrc522.py` and `pico/main.py` onto the Pico with the same names.
4. Press **Ctrl+D**. The Shell shows `READY:RFID-LAUNCHER`, then `TAG:<id>` when you hold a sticker on the reader.

### If no tags are read

Run `pico/rfid_test.py` from Thonny with **one** sticker lying on the reader and all other tags well away. It tries five receiver gain settings and tells you which value to put in `RX_GAIN` at the top of `main.py`. Some clone RC522 boards only work at the lowest gain (`0x10`), which is the default.

Close Thonny before running the launcher: only one program can use the Pico's port at a time.

## 3. Run the launcher on the PC

```
py -m pip install pyserial
cd launcher
py rfid_launcher.py
```

The launcher finds the Pico by itself and reconnects if it's unplugged. On first run it creates `tags.json`; you can also start from `tags.example.json` by copying it to `tags.json`.

It prints two addresses:

- **http://localhost:8765/manage** sets up cartridges. Scan a new sticker and a form opens for it: name, game or music, cover image, colour. Paste a Steam store link and it fills in the app ID and name.
- **http://localhost:8765** is the visitor screen. Open it on a second monitor and press F11.

![The cartridge setup page](docs/setup-page.png)

## Settings

All settings live in the `settings` section of `tags.json`.

| Setting | Default | What it does |
| --- | --- | --- |
| `serial_port` | `auto` | Set to the Pico's port (for example `COM5`) to skip auto-detection |
| `vlc_path` | `C:\Program Files\VideoLAN\VLC\vlc.exe` | Where VLC is installed |
| `vlc_hidden` | `true` | Play music without showing a VLC window |
| `pause_on_remove` | `true` | Pause music when its cartridge is removed (otherwise stop) |
| `stop_after_paused_seconds` | `60` | Stop paused music after this long |
| `stop_music_on_game` | `true` | Stop music when a game cartridge goes in |
| `game_relaunch_cooldown_seconds` | `20` | Ignore repeat scans of the same game for this long |
| `screen_port` | `8765` | Port for the visitor screen and setup page |
| `screen_host` | `127.0.0.1` | `0.0.0.0` lets a tablet on your network show the visitor screen |
| `screen_title`, `screen_subtitle`, `screen_footer` | | Text on the visitor screen |

Cartridges can only be set up from the PC running the launcher, even when `screen_host` is `0.0.0.0`.

## Troubleshooting

| Problem | Fix |
| --- | --- |
| `Couldn't load pyserial` | Run the install command the message prints. It targets the exact Python running the script |
| `tags.json has a mistake on line …` | Fix that line in Notepad; the launcher keeps using the last working version until you do |
| Browse buttons do nothing | They open a Windows file dialog that can appear behind the browser. They need tkinter, included with standard Python for Windows |
| Tags read on the bench but not in a cartridge | Thin the cartridge wall over the sticker, keep metal away, and rerun `rfid_test.py` with the sticker inside the cartridge |

## Credits

Fonts: [Silkscreen](https://fonts.google.com/specimen/Silkscreen) and [Chakra Petch](https://fonts.google.com/specimen/Chakra+Petch), used under the SIL Open Font License (see `launcher/frontend/fonts`).
