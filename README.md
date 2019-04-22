# SF Flow IOT: Case Checker

Pretty simple example here. I have devices calling in creating High Volume Platform Events. When one of my devices calls in, that might be nbd. But if that device calls in twice within an hour, that's my threshold for bad times to result, and alert all the people to do whatever. No second call? Autoclose the case.

To get here, we needed the following:
* Device_Asset__c - Yeah probably coulda just used Asset, but this was simpler. Has an ExternalID field as AssetID__c in order to map Events to.
* Device_Event__e - The actual 'event' sent from the 'device'. We listen to this in order to actually kick something off. Holds just an AssetID__c to match a record in Device_Asset__c, and a Temperature__c to have 'something' to push, even though my code does literally nothing with that value lol.
* Process to listen to the event and kick off the IoT Flow
* Process to wait an hour and auto close the Case if a second isn't fired.

I could see a need to scale this further, where I wouldn't want to be creating a case unless there truly was a need for it. I'd then keep firing to Events, but instead of searching a case, just having Apex in the Flow check the Device_Event__e bus for a matching device event in the last hour and only creating at that point. Probably a much better pattern, but this was simple enough. If you did this, and still wanted to trace *all* events, probably just start tossing em en masse into Big Objects, no fuss no muss.

## Build and Test

`git clone https://github.com/cowie/SF-IOT-DoubleCaseAlert.git`

`sfdx force:org:create -s -a IOTTwoTimes -f config/project-scratch-def.json`

`sfdx force:source:push`

Go into Process Builder, check out the Device Event Detector. This simply pushes all Platform Events to grab an assoc Device_Asset__c(mapped by AssetID) and shoots it to the Flow.

Go into Flows, check out IoT Case Creator. If a case already exists, is open and in status 'awaiting confirmation', then update the case to show it's a real case now, change to escalated, etc. No case? Create one real quick as 'awaiting confirmation'.

For the autoclosing cases, there's a Case Sword of Damacles flow that on every case insertion sets a 1h timer before hitting up Flow to delete the record. The other flow is a single item, just delete cases handed to that Flow.

## Resources

## Issues
Ehhh *should* be ok?