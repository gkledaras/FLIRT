   Copyright 2013 George N. Kledaras

   Licensed under the Apache License, Version 2.0 (the "License");
   you may not use this file except in compliance with the License.
   You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

   Unless required by applicable law or agreed to in writing, software
   distributed under the License is distributed on an "AS IS" BASIS,
   WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
   See the License for the specific language governing permissions and
   limitations under the License.

	FLIRT – Financial Language Interface for Real-time Trading

	What is FLIRT?

Financial Language Interface for Real-time Trading (FLIRT) is a modern communication protocol that is designed for performance, readability, and scalability for securities, currency or crypto-currency trading. It uses several well understood technologies such as the FIX Protocol, the global standard for electronic trading used by asset managers, brokers, and exchanges throughout the world; JavaScript Object Notation (JSON) a simple message format that is heavily used in transaction processing; and ØMQ the open, light weight, low latency transport layer that has been widely implemented by many programming languages.

	Transport Layer

The Transport Layer is defined as two separate TCP streams for inbound and out bound messages, using two separate ports. For sending messages to The Routing Gateway, you would use a Reply-Request messaging pattern. For responses back from The Routing Gateway, you would use a Publish-Subscribe messaging pattern. The messaging is done through an open source low latency project called ZeroMQ and is described below.

	ZeroMQ – ØMQ

ØMQ lightweight messaging kernel extends the standard socket interfaces with useful features. ØMQ sockets provide an abstraction of asynchronous message queues, messaging patterns, message filtering (subscriptions), seamless access to transport protocols and more. ØMQ has been ported to many different languages such as C++, Java, Python, .NET, and new languages like Scala and Go. And the various implementations in different languages can communicate with each other, making it easy to implement into whatever application is necessary for your application.

	Request-reply pattern

The request-reply pattern is used for sending requests from a client application like a trading system to one or more 
instances of The RoutingGateway. There is an immediate reply to each request sent, which can be read either synchronously or asynchronously, depending on the client trading application.
ZMQ_REQ.

This socket type allows only an alternating sequence of zmq_send(request) and subsequent zmq_recv(reply) calls. Each request sent is round-robined among all services, and each reply received is matched with the last issued request.
When a ZMQ_REQ socket enters an exceptional state due to having reached the high water mark for all services, or if there are no services at all, then any zmq_send operations on the socket shall block until the exceptional state ends or at least one service becomes available for sending; messages are not discarded.

	ZMQ_REP

A socket of type ZMQ_REP is used by a service to receive requests from and send replies back to the trading client. This socket type allows only an alternating sequence of zmq_recv(request) and subsequent zmq_send(reply) calls. Each request received is fair-queued from among all clients, and each reply sent is routed to the client that issued the last request. If the original requester doesn't exist any more the reply is silently discarded.

When a ZMQ_REP socket enters an exceptional state due to having reached the high water mark for a client, then any replies sent to the client in question shall be dropped until the exceptional state ends.

	Publish-Subscribe Pattern

Since you might get multiple or unsolicited responses from submitting trades to The Routing Gateway, FLIRT sends execution and reject information back using a publish and subscribe messaging pattern.

	ZMQ_PUB

A socket of type ZMQ_PUB is used by a publisher to distribute data. Messages sent are distributed in a fan out fashion to all connected peers. The zmq_recv(3) function is not implemented for this socket type.
When a ZMQ_PUB socket enters an exceptional state due to having reached the high water mark for a subscriber, then any messages that would be sent to the subscriber in question shall instead be dropped until the exceptional state ends. The zmq_send() function shall never block for this socket type.

	ZMQ_SUB

A socket of type ZMQ_SUB is used by a subscriber to subscribe to data distributed by a publisher. Initially a ZMQ_SUB socket is not subscribed to any messages, use the ZMQ_SUBSCRIBE option of zmq_setsockopt(3) to specify which messages to subscribe to. The zmq_send() function is not implemented for this socket type.

	System Architecture

C++/C/Python/Java Client —> JSON formatted message/ZeroMQ -> FIX Gateway -> Execution Venues, Brokers, Exchanges

	FIX Tags

The FIX (Financial Information eXchange) protocol describes a message format, tags that make up a data definition, and a session layer. As FIX was first developed over 20 years ago, the message format and session layer have been replaced by FLIRT. The data definitions are still being used since they are well understood all over the world, and the FIX committee is very good at defining new tags to support the business or trading.
FIX defines a tag and value as a pair. Tags are numbers, and values can be strings, floating point numbers, integers, dates and enumerated values. For example, a security symbol Symbol is defined as tag '55' and value as the ticker symbol 'GOOG':

	55=GOOG

This formatted will be updated in FLIRT but it will be familiar and you can use FIX tags and values as they are well understood and cared for by the FIX committee.

	Data formats

FLIRT uses a JSON payload format in order to increase interoperability between systems and processes. JSON is well understood as a format with many tools available to make it easy to bind to your application. The payload is flexible and does not need to be ordered or structured so adding tags does not require recompiles or a shared structure. There are many fast libraries that can help you parse and format JSON in the programming language of your choice.
As a rule, an integer or floating point value in FIX is represented as a number in FLIRT. Enumerated types in FIX, even if they are numbers, need to be strings to protect from the even that a new type is added as a character or string. Dates are represented as a UTC string using GMT as the FIX protocol standard. As an example:

	{ 	"35" : "D",  
		"56": "TRGA", 
		"49": "AFUND", 
		"34": 367, 
		"60": "20140321-15:21:32.524",  
		"55" : "GOOG",  
		"21": "1", 
		"54" : "1", 
		"40": "2",  
		"44" : 580.34, 
		"38" : 400  }
   
JSON also supports sending binary data such as floating point numbers and integers, minimizing the conversion from a string to number which is an expensive CPU operation.

FLIRT uses JSON for both payload data but also the return results from The Routing Gateway.

	Sending Trade Information

You submit trades to The Routing Gateway using a JSON payload and ØMQ reply request message. The return from the call is immediately returned from the Gateway; you can however wait on a thread or use the zmq_poll() function that provides a mechanism to multiplex events in a level-triggered way over a set of sockets. The return code is also a JSON formatted message and will contain a status code and the transaction time registered on our servers for your information and use.

	Receiving Trade Information

Receiving data is through a ØMQ publish-subscribe mechanism. The connection will be through a single TCP connection and the topic space will contain Target/SenderCompID, ClOrderID/OrderID and MsgType. For each order submitted, there can be multiple 

	FLIRT Messages for Trading

The Routing Gateway supports the market venues' protocol which could be FIX, OUCH or proprietary protocol. The Routing Gateway will translate FLIRT into the native protocol and maintain session connectivity. As a reference, you can use the many FIX guides on the web, or you can find the complete specifications here http://www.fixtradingcommunity.org. 

Generally in the United States, market venues use either FIX versions 4.2 or 4.4. In other parts of the world, such as Europe, FIX 4.3 and 5.0 are used. FLIRT handles the version and differences for you so you can just trade, but still have access to new features without being bound to a rigid binary format.

For trading FLIRT supports sending orders, cancels, replaces and status messages through The RoutingGateway. FLIRT supports Execution Reports for the fill information form the market venue and also any status like acknowledgements.

	Messages to TRGA

	Orders

FLIRT Orders are denoted by a MsgType (FIX tag 35) 'D' which means New Order Single. If tags such as tag "34" MsgSeqNum, "11" ClOrderID, or tag "60" TransactTime can be automatically generated by the The Routing Gateway and returned in the return string as confirmation and for your application's information.

	{ 	"35" : "D", 
		"56": "TRGA", 
		"34": 367, 
		"11": "abc477", 
		"60": "20140321-15:21:32.524",
		"55" : "GOOG",  
		"21": "1", 
		"54" : "1", 
		"40": "2",  
		"44" : 580.34, 
		"38" : 400  } 

The return from the reply is also a JSON object:

	{ 	"status" : 0, "34" : 897, "60" : "20140321-15:21:32.524" }

Errors can be denoted as the following, which loosely follow the FIX Reject message format:

	{ "status" : 1, "45" : 931, "58" : "Duplicate sequence number" }

	Cancel

FLIRT Cancels are denoted by a MsgType (FIX tag 35) 'F' and requires the ClOrderID and OrigClOrdID:

	{ 	"35" : "F",  
		"56": "TRGA",  
		"11": "abc479", 
		"41": "abc477", 
		"60": "20140321-15:21:42.361",  
		"55" : "GOOG",  
		"54" : "1",  
		"38" : 400  } 

Return is a status object like in the Order message.

	Cancel Replace

FLIRT Cancel Replaces are denoted by a MsgType (FIX tag 35) 'G' and requires the ClOrderID and OrigClOrdID. The only items that can be replaced are price and a lower quantity.

	{ 	"35" : "G",  
		"56": "TRGA", 
		"11": "abc479", 
		"41": "abc477", 
		"60": "20140321-15:21:42.632",  
		"55" : "GOOG",  
		"21": "1", 
		"54" : "1",  
		"38" : 150  }

Return is a status object like in the Order message.

	Order Status

FLIRT Order Status are denoted by a MsgType (FIX tag 35) 'H' and only requires the ClOrderID. The response to this message will come as a Execution Report (35=8) message.

	{ 	"35": "G",  
		"56": "TRGA",  
		"11": "abc479",  
		"55": "GOOG",  
		"21": "1", 
		"54": "1" } 

Return is a status object like in the Order message.

	Messages from TRGA

Messages from the The Routing Gateway come as publish subscribe pattern. The messages are Execution Reports, Cancel Reject, and system messages like Reject.

	Execution Report

FLIRT Execution Reports are denoted by a MsgType (FIX tag 35) '8' and send back the results from the FIX messages from the market venue. Execution Reports deserve careful study because they provide the main response from the end market venue regarding individual orders. As per the  FIX protocol, this message is used to acknowledge the receipt of an order including cancels/replaces, report status in response to an Order Status Request, reject orders (e.g. invalid symbol or market closed), order fill or partial fill information, and reporting of average price or commissions post-trade. Here is an example of a fill:
	{ 	"35" : "8",  
		"56": "AFUND", 
		"49": "TRGA",  
		"34": 268, 
		"37": "EGK999", 
		"11": "abc479", 
		"17": "3", 
		"20": "0", 
		"39" : "0", 
		"60": "20140321-15:21:35.124",  
		"55" : "GOOG", 
		"54" : "1", 
		"14" : 400, 
		"38" : 400, 
		"6" : 580.35, 
		"151" : 0  } 

	Execution Report – Reject

For orders, cancels or replaces that are rejected, the response is also an Execution Report but with the OrdRejReason (104) sent back:

	{ 	"35" : "8",  
		"56": "AFUND", 
		"49": "TRGA",  
		"34": 268, 
		"37": "EGK999", 
		"11": "abc479", 
		"17": "3", 
		"20": "0", 
		"39" : "2", 
		"60": "20140321-15:21:35.124",  
		"55" : "GOOG", 
		"54" : "1", 
		"14" : 400, 
		"38" : 400, 
		"6" : 580.35, 
		"151" : 0, 
		"103": 4  }
 
	Cancel Reject

Cancel Rejects are denoted by a MsgType (FIX tag 35) '9' and are sent in response for a Cancel (35=F) or Cancel Replace (35=G) that can't be honored. The reasons that a cancel cannot be honored is because it's too late to cancel, the order is unknown, or the order already pending a cancel or replace status. Here is an example:

	{ 	"35" : "9",  
		"56": "AFUND", 
		"49": "TRGA",  
		"34": 298, 
		"37": "EGK999", 
		"11": "abc479", 
		"60": "20140321-15:21:35.124", 
		"39" : 4, 
		"41": "EFF789",  
		"434" : "F" }

	Reject

Rejects are denoted by a MsgType (FIX tag 35) '3' and are reserved for non-application messages. But market venues through brokers or directly to exchanges. Reject messages will be passed through as sometimes they provide important system errors downstream, such as connection or system down status. An example follows:

	{	"35" : "3",  
		"56": "AFUND", 
		"49": "TRGA",  
		"45" : 3465, 
		"373" : 1, 
		"58" : "REQUIRED TAG 37 MISSING"  }

	Replay Messages

FLIRT provides the ability to resend messages that were missed by your application at any time, as many times as your application needs. The message requires a beginning and ending sequence number, using FIX tags BeginSeqNo (7) and EndSeqNo (16). If the ending sequence number is not known, you can send the end sequence number to 0. Non-application messages such as heartbeats or rejects will not be present, and a gap fills message will not be sent. This message will not be forwarded through to the markets, only to the FLIRT hosted gateway. Requesting messages from sequence number 2400 up to and including 2415:

	{ "replay" , "7" : 2400, "16" : 2415 }

Replaying a single message with the begin and end sequence number the same:

	{ "replay" , "7" : 325, "16" : 325 }

Replaying up to the very last known message in the gateway:    
 
	{ "replay" , "7" : 2400, "16" : 0 }
  
Be careful not to re-process any messages that have been replayed already. Any resent messages will be sent with the PossDup tag (43=Y). Also be careful not to replay too many messages at a time because it can send too many messages back to your system and can affect performance. The return is a status object, with 0 as successful:

	{ "status" : 0 }

or an error

	{ "status" : 1, "58" : "No messages found" }

	Example Code
	C++

The C++ interface is built on ØMQ for the messaging transport and rapidjson for parsing JSON messages.
Install ØMQ for your environment from http://zeromq.org/intro:get-the-software.
You can use any JSON parser or builder that you like. We chose rapidjson because it is a header library, it supports both C and C++ styles of coding, and it is very fast. You can download it from https://code.google.com/p/rapidjson/downloads/list.
Put the headers in an appropriate directory and compile and link using your favorite  C++ compiler. Adjust any link flags as required by your system:

g++ flirt.cpp  -o flirt -L/usr/local/lib -lzmq

The example can be converted to a pure C implementation as the core functions from both libraries.
#include "rapidjson/document.h"
#include "zmq/zmq.hpp"
#include "zmq/zhelpers.hpp"
#include <cstdio>
int main() {

	//** create a New Order Single as a string; you can also use the rapidjson builders which are easy to use but less efficient
	const char json[] = "{\"35\":\"D\",\"56\":\"TRGA\",\"11\":\"abc477\",\"55\":\"GOOG\",\"21\":\"1\",\"54\":\"1\",\"40\":\"2\",\"44\":580.34,\"38\":\"400\"}";
	zmq::context_t context(1);
	zmq::socket_t requester(context, ZMQ_REQ);
	requester.connect("tcp://localhost:8001");

//** send the JSON formatted Order string
	s_send(requester, json);

//** block and read the response 

	std::string string = s_recv(requester);
	std::cout << "Received reply " << requester << " [" << string << "]" << std::endl;

//** parse the status returned form the server, converting it to a JSON document using rapidjson library
	
	rapidjson::Document d;
	d.Parse<0>(string.c_str());
	printf("status=%d\n", d["status"].GetInt());
	printf("TransactTime=%s\n", d["60"].GetString());

	return 0;

}

And listening for Execution reports, Rejects and other messages fem the Market Gateway:

#include "rapidjson/document.h"
#include "zmq/zmq.hpp"
#include "zmq/zhelpers.hpp"
#include <cstdio>

int main() {

	zmq::context_t context(1);
	zmq::socket_t subscriber(context, ZMQ_SUB);
	subscriber.connect("tcp://localhost:8002");
	subscriber.setsockopt(ZMQ_SUBSCRIBE, "", 0);

//** block and read the 100 responses, then exit

	for (int update_nbr = 0; update_nbr < 100; update_nbr++)
	{

		std::string string = s_recv(subscriber);
		std::cout << "Received from MG: " << string << std::endl;
//** convert it to a rapidjson document for access to all of the fields
		rapidjson::Document d;
		d.Parse<0>(string.data());
		printf("MsgType=%s\n", d["35"].GetString());

	}

return 0;

}

	Python

You need the ØMQ (zmq) module and the json module. Python includes json starting in version 2.7. You can install the ØMQ on your Python system using:

easy_install pyzmq
or
pip install pyzmq

Please make sure these modules are installed properly before connecting to the FLIRT Market Gateway.

Here is a sample sending a request and receiving a reply used for submitting orders, cancels or replaces to the Market Gateway. Note that the recv() function will block and wait for the return status from the Market Gateway. You can instead set up a non-blocking call or run this in a separate thread, depending on your preference and trading style. While Python is a garbage collected language, you should always try to re-use memory buffers in order to minimize or even eliminate the garbage collector from running:
 
import zmq, sys, json, datetime

# ZeroMQ Context
context = zmq.Context()

# Define the socket using the "Context" and connect
sock = context.socket(zmq.REQ)
sock.connect("tcp://127.0.0.1:8001")
d = dict()
d['35'] = "D"                       # MsgType New Order
d['56'] = "TRGA"                   # TargetCompID
d['11'] = "abc477"                  # ClOrdID
d['60'] = datetime.datetime.utcnow().strftime('%Y%m%d-%H:%M:%S.%f')[:-3]   # TransactTime
d['55'] = "GOOG"                    # Symbol
d['21'] = "1"                       # Handdling Instructions, auto
d['54'] = "1"                       # Side = buy
d['40'] = "2"                       # OrderType = limit
d['44'] = 580.34                    # Limit price
d['38'] = 400                       # Quantity
# Send the message
sock.send(json.dumps(d))
# get status back
s = json.loads(sock.recv())
print "status=" + str(s['status'])
 
And to receive form the publisher, on another TCP ip and port address:
 
import sys, zmq, json, collections
# Socket to talk to server
context = zmq.Context()
socket = context.socket(zmq.SUB)
socket.connect("tcp://localhost:8002")
socket.setsockopt(zmq.SUBSCRIBE, "")
while True:
    d = json.loads(socket.recv()) # block and wait for a message to be published
    if (d['35']=='8'): # check for the FIX message type
        print "MessageType=" + d['35'] + ", symbol=" + d['55'] + ", price=" + str(d['6'])
    else:
        print "MessageType=" + d['35']
 
 

