The purpose of the project is to demostrate a new way to send virtually any information to any device through node.js hapi.js and pushbullet.

For hapi.js please visit http://hapijs.com/ for more information

For the node-pushbullet-api please read the documentation in 
		https://github.com/alexwhitman/node-pushbullet-api
for details.

Few mql4 classes has been created for easier work.


Work flow idea:

	Client									Server						Third Party
	(MT4)			msg in key-value form	(node+hapi+pushbullet)		(pushbullet, devices)
	
	//Notification class constructor called
	//Server check api is valid from pushbullet.com
	//if response is no error, notification isReady = true;
	Notfication() 	---------------------->	Listen(route)	--------->	Pushbullet
		isReady 	<----------------------	response(1) 	<---------	response(0)


	//MT4 event notification workflow
	MT4Events(route)---------------------->	Listen(route)	----------> Pushbullet 	-------> Devices
		response	<----------------------	response 		<---------- response

	//Device to MT4 (Not yet implement)
											Listen			<----------	Pushbullet	<------	Devices
											(act as device)
	MT4CheckCmd		------------------->	Listen(route)
		respones 	<-------------------	respones
		exec. cmd.

Points to note:
0.	register an account in pushbullet.com, to communicate with different devices, you can:
	a.	download pushbullet in google play store / apple app store
	b.	chrome/firefox addon, or pc app (https://www.pushbullet.com/apps) 
1.	each devices is identified by the field "iden", you can use the sample code provided to find your devices by:
	a.	replace the API-KEY in top line of index.js file
	b.	run the sample code in command prompt: node index.js
	c.	type http://localhost:8000/devices to get a list of all the devices
2.	to communicate with your devices in mt4 using either WebRequest or the header files provided

MQL4 API

1.	Dictionary (dictionary.mqh)

	This is a generic key-value pair dictionary class for easier messaging.
	a.	Format, i.e.
		key:	Symbol
		value:	EURUSD
		type:	string

		key:	Bid
		value:	1.10223
		type:	double

	b.	Usage:
		//Constructor
		void	Dictionary *info = new Dictionary();	

		//Adding key-value pair in the dictionary
		void	Dictionary.Add("key", value)

		//Get the value by key
		T 		Dictionary.Get("key");

		//Remove the key-value pair
		bool	Dictionary.RemoveByKey("key");

		//Dictionary to String
		string 	Dictionary.ToString();

		//Print value by key
		void	Dictionary.PrintByKey("key");

	c.	Example:
		Dictionary *dictionary = new Dictionary();
		
		dictionary.Add("Symbol", "EURUSD");
		dictionary.Add("Bid", 1.10223);
		
		string symbol = dictionary.Get("Symbol");
		double bid = dictionary.Get("Bid");
		
		Print(dictionary.ToString());					//Symbol|EURUSD|string;Bid|1.10223|double;

		dictionary.AppendFromString("Ask|1.10228|double");
		double ask = dictionary.Get("Ask"); //1.10228

		Print(dictionary.ToString());					//Symbol|EURUSD|string;Bid|1.10223|double;Ask|1.10228|double;

		bool result = dictionary.RemoveByKey("Symbol"); //true;
		Priint(dictionary.ToString())					//Bid|1.10223|double;Ask|1.10228|double

		dictionary.PrintByKey("Symbol"); 				//Key: Symbol not found

		Print(dictionary.Count());						//2

		Print(dictionary.GetIdxByKey("Bid"))			//0


2.	Notification (notification.mqh)
	This is the main class to communicate with the node server
	a.	Usage
		void	Notification(string url, string key)	//url: node server url, key: pushbullet api-key
														//if any error, Notification's member isReady will set to false. in case isReady = false, no further task will perform;
		
		void	AddDevice(string device);				//add exist device listed in pushbullet.com into the system

		int		Send(string route, string msg, string device = NULL);		
														//send a string msg to a device, if no device is specified, system will attemp to send to devices previously added by AddDevice

		int 	Send(string route, Dictionary dictionary, string device=NULL);
														//send a message with additional information like a screen capture file stored in the dictionary
