lognormalizer (liblognorm-1.1.0)
=======================
(Installed by liblognorm) 

## Example Usage:

Running *lognormalizer* to test your rulebase manually.

```shell
$ lognormalizer -r ./example.rulebase -e json < ./example.log
{"src-port": "14121", "src-ip": "192.168.0.1", "username": "bobuser"}
```



## Usage Flags

```
user@sensor:/var/log/passivedns # lognormalizer --help
lognormalizer: illegal option -- -
Options:
    -r<rulebase> Rulebase to use. This is required option
    -e<json|xml|csv>
                 Change output format. By default, Mitre CEE is used
    -E<format>   Encoder-specific format (used for CSV, read docs)
    -T           Include 'event.tags' in JSON format
    -oallowRegex Allow regexp matching (read docs about performance penalty)
    -p           Print back only if the message has been parsed succesfully
    -t<tag>      Print back only messages matching the tag
    -v           Print debug. When used 3 times, prints parse tree
    -d           Print DOT file to stdout and exit
    -d<filename> Save DOT file to the filename
```

## Observed Difference between `sagan` and `lognormalizer`

WARNING: When running lognormalizer the space between "rule=: %" and "rule=:%" is parsed!! so change when testing..
