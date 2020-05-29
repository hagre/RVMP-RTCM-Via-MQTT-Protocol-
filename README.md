# RVMP-RTCM-Via-MQTT-Protocol

RVMP (RTCM Via MQTT Protocol) V0.1 (first very basic approach, but some features for future are in preperation).

In use by RVMT (RTCM Via MQTT Transmitter - https://github.com/hagre/RVMT-RTCM-VIA-MQTT-TRANSMITTER) using the MQTT msg/protocol (as a secure and opensource alternative to NTRIP) to get RTK correction data from BaseStations to Broker and to Rover GPS units.

Basicly, this is a try to implement something like a NTRIP caster with MQTT features. There are a lot of similarities to NTRIP (V1.0/V2.0), because it is a knowen and usefull protocol with most needet features for this task.

Attension: Topics are casesensitive!!

# Topics:

# Basestation (MNTP) publish:
Welcome msg: (transmitted when a new connection is established, or a feature is updated, .. )
+ RTK/Base/<NAME of BASE>/Status/Tversion/ - e.g "2.2" - Version of transmitter firmware [QOS = 1, retain = true, last_will = null] [read acces only by caster app, read/write by Basestation ]
+RTK/Base/<NAME of BASE>/Status/Pversion/ - e.g "0.1" - Version of protocol [QOS = 1, retain = true, last_will = null] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/<NAME of BASE>/Status/RTCM/<TYPE of MSG>/ - e.g "1", "0.5" or "30" - time in seconds between update (~frequenz) [QOS = 1, retain = true, last_will = null] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/<NAME of BASE>/Status/Operation/ - "ON" full working RTCM, "STBY" receiving RTCM, but not transmitting over MQTT, "ERROR" having some trouble, "OFF" - not avaliable  [QOS = 1, retain = true, last_will = "OFF"] [read acces only by caster app, read/write by Basestation ]
+ RTK/Base/<NAME of BASE>/Status/Error/ - type of error e.g "NO_RTCM_INPUT"  [QOS = 1, retain = false, last_will = null] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to implement: RTK/Base/<NAME of BASE>/Status/Position/Lat/ - LatPosition [QOS = 1, retain = true, last_will = null] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to implement: RTK/Base/<NAME of BASE>/Status/Position/Long/ - LongPosition [QOS = 1, retain = true, last_will = null] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to try: RTK/MNTP/NAME of BASE/Command/Out/PW/ - e.g "Password" [QOS = 1, retain = false, last_will = null] [read acces only by caster app, read/write by Basestation ] //further work required 
+ //planning to try: RTK/MNTP/NAME of BASE/Command/Out/USER/ - e.g "Username" (? username == <NAME of MNTP>) [QOS = 1, retain = false, last_will = null] [read acces only by caster app, read/write by Basestation ] //further work required

for each RTCM msg type:
+ RTK/Base/NAME of BASE/RTCM/<TYPE of MSG>/ [retain = false, last_will = null] [read acces only by caster app, read/write by Basestation ]

# Basestation (MNTP) subscribe:
+ RTK/Base/<NAME of BASE>/Command/In/RTCM/ 
+ RTK/Base/<NAME of BASE>/Command/In/GPS/ 
+ RTK/Base/<NAME of BASE>/Command/In/Transmitter/ 
+ //planning to try: RTK/Base/<NAME of BASE>Command/In/Serial/
+ //planning to try: RTK/Base/<NAME of BASE>/Serial/In/ 

# Caster subscribe:
+ RTK/Base/<NAME of BASE>/Status/Tversion/ 
+ RTK/Base/<NAME of BASE>/Status/Pversion/ 
+ RTK/Base/<NAME of BASE>/Status/RTCM/<TYPE of MSG>/ 
+ RTK/Base/<NAME of BASE>/Status/Operation/ 
+ RTK/Base/<NAME of BASE>/Status/Error/
+ RTK/Base/<NAME of BASE>/Status/Position/Lat/
+ RTK/Base/<NAME of BASE>/Status/Status/Position/Long/
+ //planning to try: RTK/MNTP/<NAME of BASE>/Command/Out/PW/
+ //planning to try: RTK/MNTP/<NAME of BASE>/Command/Out/USER/
+ RTK/Base/<NAME of BASE>/RTCM/#

some calculation in the caster ....  
- acvtivate or deactivate Basestations
- solve errors witch reboot if required
- copy RTCM from RTK/Base/<NAME of BASE>/... to RTK/MNTP/<NAME of MNTP>
- keep track of the Basestations
- construct and update Mountpointlist (all possible avaliable Basestations )
- keep track of the Basestations

# Caster publish:
+ RTK/MNTP/<NAME of MNTP>/RTCM/<TYPE of MSG>/ [retain = false, last_will = null] [read acces by all RTK users, read/write by caster]
commands to Basestation
+ RTK/MNTP/<NAME of MNTP>/Command/In/RTCM/ - "ON" or "OFF" to activate/deactivate RTCM transmission if required [QOS = 1, retain = false, last_will = null] [read/write acces by caster app, read by correct Basestation ]
+ RTK/MNTP/<NAME of MNTP>/Command/In/GPS/ - "REBOOT" to force an GPS reboot [retain = false, last_will = null] [QOS = 1, read/write acces by caster app, read by correct Basestation ]
+ RTK/MNTP/<NAME of MNTP>/Command/In/Transmitter/ - "REBOOT" to force an transmitter reboot (e.g ESP32 modul) [QOS = 1, retain = false, last_will = null] [read/write acces by caster app, read by correct Basestation ]
+ RTK/MNTPList/<NAME of BASE>/Status/Pversion/ [QOS = 1, retain = true, last_will = null] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/<NAME of BASE>/Status/RTCM/<TYPE of MSG>/  [QOS = 1, retain = true, last_will = null] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/<NAME of BASE>/Status/Operation/  [QOS = 1, retain = true, last_will = null] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/<NAME of BASE>/Status/Position/Lat/ [QOS = 1, retain = true, last_will = null] [read/write acces by caster app, read by clients ]
+ RTK/MNTPList/<NAME of BASE>/Status/Position/Long/ [QOS = 1, retain = true, last_will = null] [read/write acces by caster app, read by clients ]

# Clients publish:
+ RTK/User/Subscribe/<NAME of USER>/ - "<NAME of MNTP>" to request MNTP transmission, caster will provide needet data if on the mountpointlist [QOS = 1, retain = false, last_will = "ByBy"] [read/write acces by clients app, read by caster ]

# Clients subscribe:
initially and refresh if needed
+ RTK/MNTPList/# - get complete list
or 
+ RTK/MNTPList/<NAME of MNTP>/# - get single desired MNTP
continously 
+ RTK/MNTP/<NAME of MNTP>/Status/Operation/ - check operation of desired MNTP
+ RTK/MNTP/<NAME of MNTP>/RTCM/# - get all RTCM msg avaliable
or 
+ RTK/MNTP/<NAME of MNTP>/RTCM/<TYPE of MSG>/ - just for each desired msg type

# //MaintenanceClient subscribe:
+ //planning: RTK/Base/<NAME of BASE>/Serial/Out/

# //MaintenanceClient publish:
+ //planning: RTK/Base/<NAME of BASE>/Serial/In/ -  Serial transmission to GPS if required (config of chip) [retain = false, last_will = null] //serial transmission will be included in a later version
+ //planning to try: RTK/Base/<NAME of BASE>Command/In/Serial/ - "ON" or "OFF" to activate/deactivate Serial transmission to GPS if required (config of chip) [retain = false, last_will = null] //serial transmission will be included in a later version


# Hints:
MNTP - Mountpoint
<NAME of MNTP>  == <NAME of BASE>- e.g. XYZ01
<TYPE of MSG> - e.g. 1074


# further plans:
+add PW acces for Basestation (if required at all)
+add a special client ID  (user "BaseMaintenance") for mantenance my basestations with serial com
+serial com with base (RTK/Base/<NAME of BASE>Command/In/Serial/ ...)
in far future
+ RTK/User/Subscribe/<NAME of USER>/Position/
+ and construckt/calculate virtual basestation on user request for user position - give virtual name MNTP and publish accordingly
