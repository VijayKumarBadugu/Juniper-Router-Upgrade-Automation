# Juniper-Router-Upgrade-Automation
Author        	 : 	Vijay Kumar Badugu
                	vbadugu@juniper.net

Date          	 : 	08/6/2015.

Description   	 : 	The purpose of the of this project is to automate BR router code upgrade process for Verizon.
			Verizon does router upgrades regularly which involves in removal of the traffic, upgrading the JUNOS 
			code and addition of the traffic post upgrade.

			The entire process for BR code upgrade is automated using two scripts. They are:
				a) Upgrade.slax
				b) Traffic_Restoral.slax				


Pseudo Code 	:
 
Upgrade.slax	:	Upgrade.slax script does the preliminary checks,saves the configuration and removes the traffic.
			Here is the  pseudo code for Upgrade.slax.
			
		1) 	Save the existing configuration in both routing engines at "/var/home/full/"

		2) 	Search for available Junos codes at "/var/home/full/" .Prompt the user to choose required code
	
		3) 	If the required Junos code is not available search for available codes in other routing engine.
	
		4) 	After the user selects the required Junos code make sure that the selected Junos code is staged in both routing 
			engines
	
		5) 	Check whether the router is booted from flash disk . Use "show system storage" and ensure the router properly 
			booted from the flash disk.This would be indicated by “ad0xxx” mounted on “/”.
			( JUNOS 13-3R3-S8 - MX - Code Upgrade document,chapter 5.2)
		
		6) 	Ensure the “compact flash” and “disk” are in the bootlist .Use "sysctl –a | grep bootdev" and 
			interrupt the program if “compact flash” and “disk” are not present in the bootlist.
	
		7) 	Remove the traffic .Follow "gdnm-1125 (1)" document.

		8) 	Check whether VOIP traffic by running " show aps ", " show bgp summary| match 1661 ",
			" show bfd session | match ^152 " , " show vrrp ".

		9) 	If VOIP traffic not present .First Deactivate External BGP Neighbors.

		10) 	Check for KRT queue for every 60 secs . Proceed to next step if you get queue empty two consequtive times.

		11) 	Deactivate routing instances if already present pre maintenance.

		12)	Set ISIS in overload.

		13)	Deactivate BGP protocols.

		14)	Check for KRT queue for every 60 secs . Proceed to next step if you get queue empty two consequtive times.
	
		15)	Deactivate protocols.
	
		16) 	Deactivate routing options if already present pre maintenance.

		17)	Deactivate chassis redundancy .

		18)	Run "request system snapshot" on maintenance routing engine . Check whether it returns any error.
	
		19)	Run "request system snapshot" on other routing engine . Check whether it returns any error.

		20)	Give out the commands to do code upgrade for both routing engines .



Traffic_Restoral.slax	: Traffic_Restoral.slax scripts restores router to pre maintenance configuration.
				Here is the  pseudo code for Traffic_Restoral.slax.
			
		1) 	First check whether both routing engines are upgraded to same JUNOS code.
	
		2)	Check whether Re0 is configured as "master" else run " request chassis routing-engine master switch " to 
			make re0 as master . Interrupt the script if re0 is not configured as master.

		3)	Activate routing-options if routing-options are deactivated pre maintenance.

		4)	Activate chassis redundancy .

		5)	Activate protocols.
	
		6)	Check for KRT queue for every 60 secs . Proceed to next step if you get queue empty two consequtive times.

		7)	Activate BGP.

		8)	Check for KRT queue for every 60 secs . Proceed to next step if you get queue empty two consequtive times.

		9)	Restore Pre Maintenance configuration of  ISIS.

		10)	Activate routing-instances.

		32)	Activate External BGP neighbors which were deactivated pre maintenance.

		33) 	List out all the differences of pre maintenance and post maintenance  of all the below outputs
			
			a)		show chassis fpc pic-status
			b)		show isis Adjacency
			c)		show bgp summary
			d)		show ldp neighbor
			e)		show ldp session
			f)		show bfd session
			g)		show interface description
			i)		show mpls interfaces
			j)		show rsvp interfaces
			k)		show chassis alarms
			l)		show mpls terse
			m)		show route summary
			n)		show chassis environment
