---
title: "Burp Suite Extension : Raw Collaborator"
date: 2023-08-T05:00:00-01:00
categories:
  - blog
tags:
  - BurpSuite
toc: True
toc_label: BurpSuite
toc_icon: "bug"
---

## Burp Suite Extension : Raw Collaborator

The following describes the set up and usage of a Burp Suite Extension I wrote, that creates a new Collaborator URL and dumps the raw Interaction transactions as text to the console of the plugin, or allows the log to be written to a file on the system that can be further analyzed.

This plugin can prove itself to be useful when large number of Burp Suite Collaborator interactions need to be monitored and scraped as plain text from what Collaborator provides. The JSON based results obtained can further be pa

### Create Extension File
Create a local `raw_collaborator.py` file and save it with the following code :
```python
__author__ = 'notnotnotveg (https://wiki.notveg.ninja)'
__date__ = '20230301'
__version__ = '1.0'
__description__ = """\
A Burp Suite Extension that creates a new Collaborator URL 
and dumps the raw Interaction transactions as text to the 
console of the plugin, or allows the log to be written to 
a file on the system that can be further analyzed.
"""

from burp import IBurpExtender, IBurpCollaboratorInteraction, IBurpCollaboratorClientContext

class BurpExtender(IBurpExtender, IBurpCollaboratorInteraction, IBurpCollaboratorClientContext ):
	def registerExtenderCallbacks(self, callbacks):
		self._callbacks = callbacks
		self._helpers = callbacks.getHelpers()
		callbacks.setExtensionName("Raw Collaborator")

		self.collaboratorContext = callbacks.createBurpCollaboratorClientContext()
		print( "{'_Collaborator_URL' : '" + self.collaboratorContext.generatePayload(True) + "'}")
		while True:
			collresult = self.collaboratorContext.fetchAllCollaboratorInteractions()
			for coll in collresult:
				interaction_raw = str(coll.getProperties())
				interaction = str.replace(interaction_raw,"u'","'")
				self._callbacks.printOutput(interaction)
		return

```

### Load Extension
Navigate to Extensions > Installed > Burp Extensions and click on Add.
Select the Python file for the Extension and select a file to save the output.
![2023-08-05-burpsuite_rawcollab-1](https://github.com/notnotnotveg/notnotnotveg.github.io/assets/65092714/0d23848a-f32c-487d-8654-5f023f456a95)



A new Extension called "Raw Collaborator" should now get added to the list.
### Test the Extension

One the extension is loaded, a new Collaborator URL gets automatically created and printed on the Output of the Extension. As an example : 
![2023-08-05-burpsuite_rawcollab-2](https://github.com/notnotnotveg/notnotnotveg.github.io/assets/65092714/72db8d07-6336-4d12-9127-4cce5f3618fc)


A sample way to test the plugin by making DNS interactions to it using bash : 
```
for num in {1..100} ;  do  dig +short CNAME $num.RANDOM-IDENTIFIER.oastify.com; done
```

The log file configured should contain logs as follows :
```
{'_Collaborator_URL' : 'RANDOM-IDENTIFIER.oastify.com'}
{'client_port': '25491', 'time_stamp': '2023-Aug-20 16:38:57.540 UTC', 'interaction_type': 'collaborator', 'client_ip': '10.0.0.1', 'query_type': 'CNAME', 'interaction_id': 'RANDOM-IDENTIFIER', 'type': 'DNS', 'raw_query': 'BASE64-ENCODED-STR'}
```

To create a new Collaborator URL, Unload and Load the extension back again.
