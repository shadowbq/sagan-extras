Normalizer (0.3.4)
(Installed by liblognorm) 
=======================

$ normalizer -r ./messages.sampdb -ejson < ./messages.log > messages-normalized.log

-r = path to the rulebase

-e = output format [default: syslog]
       [available: xml, json, syslog, csv]

-E = Fields that should be dispended [default: all] 
       (-E “host tag” -> that only dispend the host and the tag field) 

-p = just the parsed messages will be dispensed 

-v = debug outout 
       (-v is the normal debug mode; -vv is an expanded debug mode with more information)

-d = dot file 
       (Is used for creating a graph of the rulebase)
-T = Flat tags

-t = Fields that should be mandatory, only messages with these defined fields will be outputed.
