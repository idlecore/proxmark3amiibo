# Proxmark 3 Iceman Amiibo Scripts

This repo contains minor changes to the amiibo scripts packaged with Iceman. It corrects errors with the placement of temp files on windows and allows these scripts to work correctly with the Proxmark3 Easy build from [proxmarkbuilds.org](https://proxmarkbuilds.org/).

The Latest RRG build does not support python scripts, so the hf_mfu_amiibo_restore.lua script has been updated to make an "os.execute" call rather than go through the proxmark core.

## Dependancies

- pyamiibo - Can be installed with "pip install pyamiibo"
- Latest RRG Iceman build from [proxmarkbuilds.org](https://proxmarkbuilds.org/)

## Usage

1. Copy scripts from this repo into your proxmark/client folder.
    - Enusure each script is in the correct folder.
    - *NOTE*: The pyscripts folder probably won't exist and will have to be created.
2. Run `script run hf_mfu_amiibo_restore.lua -f <amiibo_bin> -k <key>`
    - Ensure your hf tag is placed on the hf antenna prior to running this command. It will imediately start writing.

## Change Details

```diff
--- a/../hf_mfu_amiibo_restore.lua
+++ b/./luascripts/hf_mfu_amiibo_restore.lua
@@ -125,15 +125,18 @@ local function main(args)
         return oops("Can't read UID of NTAG215 card.  Reposition card and try again.")
     end
 
-    local tmp = ('%s.bin'):format(os.tmpname())
-    local amiibo_file = io.open(tmp, 'w+b')
+    local tmp = '\\tmp' .. ('%s.bin'):format(os.tmpname())
+    print(tmp)
+    local amiibo_file, err = io.open(tmp, 'w+b')
+    print(err)
     amiibo_file:write(bin.pack('H', hex))
     amiibo_file:close()
-    local tmp2 = ('%s.bin'):format(os.tmpname())
+    local tmp2 = '\\tmp' .. ('%s.bin'):format(os.tmpname())
 
     print('generating new Amiibo binary for NTAG215 '..ansicolors.green..uid)
     core.clearCommandBuffer()
-    core.console(('script run amiibo_change_uid %s %s %s %s'):format(uid, tmp, tmp2, core.search_file('resources/key_retail', '.bin')))
+    os.execute(('python pyscripts/amiibo_change_uid.py %s %s %s %s'):format(uid, tmp, tmp2, core.search_file('resources/key_retail', '.bin')))
+    -- core.console(('script run amiibo_change_uid %s %s %s %s'):format(uid, tmp, tmp2, core.search_file('resources/key_retail', '.bin')))
 
     -- let's sanity check the output
     hex, err = utils.ReadDumpFile(tmp2)
```