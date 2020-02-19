# DART Util

The CLI tool to inspect DART database, and write to it directly.

```
           --version display the version
-dart --dartfilename Sets the dartfile: default /tmp/default.drt
   -i    --inputfile Sets the HiBON input file name
   -o   --outputfile Sets the output file name
              --from Sets from angle: default full
                --to Sets to angle: default full
  -fn   --useFakeNet Enables fake hash test-mode: default false
              --read Excutes a DART read sequency: default false
               --rim Performs DART rim read: default false
            --modify Excutes a DART modify sequency: default false
               --rpc Excutes a HiPRC on the DART: default false
          --generate Generate a fake test dart
              --dump Dumps all the arcvives with in the given angle
   -w        --width Sets the width and is used in combination with the generate
   -r        --rings Sets the rings and is used in  combination with the generate
        --initialize 
   -h         --help This help information.
```