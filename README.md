# JavaScript – Events Bubbling – Illustrated

Tutorial on “Events Bubbling” in JavaScript language

***Abstract: This is tutorial text on “events bubbling” in JavaScript. Some theory is explained, and several JavaScript examples are shown.***     

## 1 Introduction
The purpose of this article is to provide an illustrative example of the “bubbling and capturing of events” in browser DOM. This is not meant to be a complete tutorial, but rather a “proof of concept” of how events propagate in DOM.   

## 2 Theoretical background
Every DOM Node can generate an event. An event is a signal that something happened.
To react to an event, we need to assign a Node an ***event handler***. There are 3 ways to assign a handler:   

1. 	**Html attribute usage**. For example: onclick=”myhandler(e)”;   
1. 	**DOM property usage.** For example: element.onclick=myhandler;   
1. 	**Javascript method.** For example: element.addEventListener(e, myhandler, phase);    

The existence of the method “addEventListener” is coming from the fact that every Node in DOM inherits from EventTarget [2] which acts as a root abstract class. So, every Node can receive events and react on them via an event handler.    

Most events in DOM propagate. Typically there are 3 phases of event propagation:   

1. 	**Capturing phase.** Propagation from window object toward specific target event   
1. 	**Target phase.** The event has reached its target.   
1. 	**Bubbling phase.** Propagation from the target toward the window object   

An ***event object*** is being thrown when an event happens. It contains properties describing that event. Two properties are very interesting to us:    

- 	***event.currentTarget*** – object that handled the event (for example, some parent of an element that was actually clicked, that got that event by the act of bubbling)   
- 	***event.target*** – target element that initiated the event ( for example, the element on which it was actually clicked)    


## 3 Example01 -Simple event propagation demo   
### 3.1 Going over all Nodes   
First, we want to create a list of all Event recipients of interest. That will include all Nodes in our DOM plus “document” and “window” objects. Here is the code for that:    

```js
function getDescendants(node, myArray = null) {
	//note that we are looking for all Nodes, not just Elements
	//going recursively into dept
	var i;
	myArray = myArray || [];
	for (i = 0; i < node.childNodes.length; i++) {
		myArray.push(node.childNodes[i])
		getDescendants(node.childNodes[i], myArray);
	}
	return myArray;
}

function CreateListOfEventRecipients() {
	let result;
	//get all Nodes inside document
	result = getDescendants(document);
	//add Document object
	result.push(window.document);
	//add Window object
	result.push(window);
	return result;
}
```

### 3.2 Logging Events   
Next, we want to nicely log events in our event-handler function. Here is the code for that:   

```js
function EventDescription(e, handlerPhase) {
	//function to log info from event object
	//here are properties, that are important
	//->handlerPhase -> we log info from which handler this event is coming 
	//                  from, is this from bubbling of capturing handler
	//->e.toString() -> class of event object
	//->e.type -> type of even, for example "click"
	//->e.timeStamp -> timestamp of event in milliseconds,
	//                 counted from load of the page
	//->e.target -> real target (object) of event, for example
	//              real object that was clicked on
	//->e.currentTarget -> current target (object) that is throwing 
	//                     this event, since event got here by either
	//                     bubbling of capturing propagation
	//->e.eventPhase -> real phase of event, regardless of handler that
	//                  created event handler, can be 1-capturing, 
	//                  2-target, 3-bubbling phase
	/* Sample execution:
	6600.900000095367[object PointerEvent]-----------
	......EventType:click..HandlerPhase:Capturing..EventPhase:1 (Capturing)
	......Target:[object HTMLElement], nodeName:B, id:Bold4
	......CurrentTarget:[object HTMLDivElement], nodeName:DIV, id:Div1
	*/
	const dots = "......";
	let result;
	if (e !== undefined && e !== null) {
		let eventObject = e.toString();
		let eventType = (e.type) ?
			(e.type.toString()) : undefined;
		let eventTimestamp = (e.timeStamp) ?
			(e.timeStamp.toString()) : undefined;
		let eventTarget = (e.target) ? ObjectDescription(e.target) : undefined;
		let eventCurrentTarget = (e.currentTarget) ?
			ObjectDescription(e.currentTarget) : undefined;
		let eventPhase = (e.eventPhase) ?
			PhaseDescription(e.eventPhase) : undefined;

		result = "";
		result += (eventTimestamp) ? eventTimestamp : "";
		result += (eventObject) ? eventObject : "";
		result += "-----------<br>";
		result += dots;
		result += (eventType) ? ("EventType:" + eventType) : "";
		result += (handlerPhase) ? ("..HandlerPhase:" + handlerPhase) : "";
		result += (eventPhase) ? ("..EventPhase:" + eventPhase) : "";
		result += "<br>";
		result += (eventTarget) ? (dots + "Target:" + eventTarget + "<br>") : "";
		result += (eventCurrentTarget) ? (dots + "CurrentTarget:" +
			eventCurrentTarget + "<br>") : "";
	}
	return result;
}  
```

### 3.3 Full Example01 code   
Here is the full Example01 code, since most people like code that they can copy-paste.   

```js   
<!DOCTYPE html>
<html>

<body>
    <!--Html part---------------------------------------------->
    <div id="Div1" style="border: 1px solid; padding:10px; 
        margin:20px; background-color:aqua;">
        Div1
        <div id="Div2" style="border: 1px solid; padding:10px; 
            margin:20px;background-color:chartreuse">
            Div2
            <div id="Div3" style="border: 1px solid; padding:10px;
                margin:20px; background-color:yellow; ">
                Div3<br/>
                Please click on 
                <b id="Bold4" style="font-size:x-large;">bold text</b> only.
            </div>
        </div>
    </div>
    <hr />
    <h3>Example 01</h3>
    <br />
    <button onclick="task1()"> Task1-List of all nodes</button><br />
    <br />
    <button onclick="task2()"> Task2-Activate EventHandlers</button><br />
    <br />
    <button onclick="task3()"> Task3-Clear Output Box (delayed 1 sec)</button><br />
    <hr />

    <h3>Output</h3>
    <div id="OutputBox" style="border: 1px solid; min-height:20px">
    </div>

    <!--JavaScript part---------------------------------------------->
    <script>

        function OutputBoxClear() {
            document.getElementById("OutputBox").innerHTML = "";
        }

        function OutputBoxWriteLine(textLine) {
            document.getElementById("OutputBox").innerHTML
                += textLine + "<br/>";
        }

        function OutputBoxWrite(textLine) {
            document.getElementById("OutputBox").innerHTML
                += textLine;
        }

        function ObjectDescription(oo) {
            //expecting Node or Window or Document
            //logging some basic info to be able to 
            //identify which object is that in the DOM tree
            let result;

            if (oo != null) {
                result = "";
                result += oo.toString();

                if (oo.nodeName !== undefined) {
                    result += ", nodeName:" + oo.nodeName;
                }

                if (oo.id !== undefined && oo.id !== null
                    && oo.id.trim().length !== 0) {
                    result += ", id:" + oo.id;
                }

                if (oo.data !== undefined) {
                    let myData = oo.data;
                    let length = myData.length;
                    if (length > 30) {
                        myData = myData.substring(0, 30);
                    }
                    result += `, data(length ${length}):` + myData;
                }
            }

            return result;
        }

        function PhaseDescription(phase) {
            //some text to explain phase numbers
            let result;
            if (phase !== undefined && phase !== null) {
                switch (phase) {
                    case 1:
                        result = "1 (Capturing)";
                        break;
                    case 2:
                        result = "2 (Target)";
                        break;
                    case 3:
                        result = "3 (Bubbling)";
                        break;
                    default:
                        result = phase;
                        break;
                }
            }
            return result;
        }

        function EventDescription(e, handlerPhase) {
            //function to log info from event object
            //here are properties, that are important
            //->handlerPhase -> we log info from which handler this event is coming 
            //                  from, is this from bubbling of capturing handler
            //->e.toString() -> class of event object
            //->e.type -> type of even, for example "click"
            //->e.timeStamp -> timestamp of event in milliseconds,
            //                 counted from load of the page
            //->e.target -> real target (object) of event, for example
            //              real object that was clicked on
            //->e.currentTarget -> current target (object) that is throwing 
            //                     this event, since event got here by either
            //                     bubbling of capturing propagation
            //->e.eventPhase -> real phase of event, regardless of handler that
            //                  created event handler, can be 1-capturing, 
            //                  2-target, 3-bubbling phase
            /* Sample execution:
            6600.900000095367[object PointerEvent]-----------
            ......EventType:click..HandlerPhase:Capturing..EventPhase:1 (Capturing)
            ......Target:[object HTMLElement], nodeName:B, id:Bold4
            ......CurrentTarget:[object HTMLDivElement], nodeName:DIV, id:Div1
            */
            const dots = "......";
            let result;
            if (e !== undefined && e !== null) {
                let eventObject = e.toString();
                let eventType = (e.type) ?
                    (e.type.toString()) : undefined;
                let eventTimestamp = (e.timeStamp) ?
                    (e.timeStamp.toString()) : undefined;
                let eventTarget = (e.target) ? ObjectDescription(e.target) : undefined;
                let eventCurrentTarget = (e.currentTarget) ?
                    ObjectDescription(e.currentTarget) : undefined;
                let eventPhase = (e.eventPhase) ?
                    PhaseDescription(e.eventPhase) : undefined;

                result = "";
                result += (eventTimestamp) ? eventTimestamp : "";
                result += (eventObject) ? eventObject : "";
                result += "-----------<br>";
                result += dots;
                result += (eventType) ? ("EventType:" + eventType) : "";
                result += (handlerPhase) ? ("..HandlerPhase:" + handlerPhase) : "";
                result += (eventPhase) ? ("..EventPhase:" + eventPhase) : "";
                result += "<br>";
                result += (eventTarget) ? (dots + "Target:" + eventTarget + "<br>") : "";
                result += (eventCurrentTarget) ? (dots + "CurrentTarget:" +
                    eventCurrentTarget + "<br>") : "";
            }
            return result;
        }


        function getDescendants(node, myArray = null) {
            //note that we are looking for all Nodes, not just Elements
            //going recursively into dept
            var i;
            myArray = myArray || [];
            for (i = 0; i < node.childNodes.length; i++) {
                myArray.push(node.childNodes[i])
                getDescendants(node.childNodes[i], myArray);
            }
            return myArray;
        }

        function CreateListOfEventRecipients() {
            let result;
            //get all Nodes inside document
            result = getDescendants(document);
            //add Document object
            result.push(window.document);
            //add Window object
            result.push(window);
            return result;
        }

        function MyEventHandler(event, handlerPhase) {
            OutputBoxWrite(EventDescription(event, handlerPhase));
        }

        function BubblingEventHandler(event) {
            MyEventHandler(event, "Bubbling");
        }

        function CapturingEventHandler(event) {
            MyEventHandler(event, "Capturing");
        }

        window.onerror = function (message, url, line, col, error) {
            OutputBoxWriteLine(`Error:${message}\n At ${line}:${col} of ${url}`);
        };

        function task1() {
            //we need to delay for 1 second aaa 
            //to avoid capturing click on button itself 
            setTimeout(task1_worker, 1000);
        }
        function task1_worker() {
            //show all Nodes plus Window plus Document 
            //they all receive events
            OutputBoxClear();
            OutputBoxWriteLine("Task1");
            let arrayOfEventRecipientCandidates = CreateListOfEventRecipients();

            for (let i = 0; i < arrayOfEventRecipientCandidates.length; ++i) {
                let description = ObjectDescription(arrayOfEventRecipientCandidates[i]);
                OutputBoxWriteLine(`[${i}] ${description}`);
            }
        }

        function task2() {
            OutputBoxClear();
            OutputBoxWriteLine("Task2");
            //we need to delay for 1 second creation of EventsHandlers
            //to avoid capturing click on button Task2 itself 
            setTimeout(task2_worker, 1000);
        }

        function task2_worker() {
            //create list of all Event Recipients
            //and assigning Events Handlers
            let arrayOfEventRecipientCandidates = CreateListOfEventRecipients();

            for (let i = 0; i < arrayOfEventRecipientCandidates.length; ++i) {
                //we check that each member of list can receive events
                if ("addEventListener" in arrayOfEventRecipientCandidates[i]) {
                    //adding Event Handlers for Bubbling phase
                    //actually, this will be handler for bubbling and target phase
                    arrayOfEventRecipientCandidates[i].addEventListener("click", BubblingEventHandler);
                    //adding Event Handlers for Capturing phase
                    //actually, this will be handler for capturing and target phase
                    arrayOfEventRecipientCandidates[i].addEventListener("click", CapturingEventHandler, true);
                }
                else {
                    //here printout if any object from the list can not receive Events
                    let description = "Object does not have addEventListener:" +
                        ObjectDescription(arrayOfEventRecipientCandidates[i]);
                    OutputBoxWriteLine(`[${i}] ${description}`);
                }
            }
        }

        function task3() {
            //we need to delay for 1 second aaa 
            //to avoid capturing click on button Task3 itself 
            setTimeout(OutputBoxClear, 1000);
        }

    </script>
</body>
<!--
Output

    -->

</html>
```

### 3.4 Application screenshot   
Here is what the application looks like:   

![39_31pic.png](Readme/39_31pic.png)

### 3.5 Execution – finding all Event-Targets   
First, we will show all Event Targets that our methods find. That includes all Nodes plus “document’ and “window” objects. There are around 60 objects on that list. Note that it includes ALL Nodes, including text and comment nodes.     

![39_32pic.png](Readme/39_32pic.png)

### 3.6 Execution – activating Event-Handlers   
Then we activate Event-Handlers for all objects in the list.   

### 3.7 Execution – Click Event   
Now we do our click. Here is a log of events from the application:   

![39_34pic.png](Readme/39_34pic.png)

## 3.8 Comments   
For those who like DOM tree diagrams, here is a diagram of what happened in the application.   


![39_35pic.png](Readme/39_35pic.png)

•	Please note that the above diagram does not include “document” and “window” objects that, as can be seen from the log, are also recipients of the click event   
•	Please note that although we actually clicked text Node **“#text-6”**, that Node did not receive the event, but the Element that contains it **“B Id:Bold4”** did receive the click event.    

## 4 Example02

The reason I created this example is because I saw claims on the Internet that in the Target phase, events and not always run in Capturing-Bubbling order, but in order they are defined in the code. I find those claims untrue, at least for this version of Chrome I used for testing.    

### 4.2 Code   
I will not put here whole code, since is similar to the previous example. Here are just key parts:   

```js
function task1() {
	OutputBoxClear();
	OutputBoxWriteLine("Task1-Activate EventHandlers");

	//Div1 event handlers
	let div1=document.getElementById("Div1");
	div1.addEventListener("click", (e)=>MyEventHandler(e, "Bubbling"));
	div1.addEventListener("click", (e)=>MyEventHandler(e, "Capturing"), true);

	//Div2 event handlers
	let div2=document.getElementById("Div2");
	div2.addEventListener("click", (e)=>MyEventHandler(e, "Bubbling"));
	div2.addEventListener("click", (e)=>MyEventHandler(e, "Capturing"), true);

	//Div3 event handlers
	//we deliberately mix defining order of bubbling and capturing events
	//to see order when events are fired (**)
	let div3=document.getElementById("Div3");
	//actually, this will be handler for bubbling and target phase
	div3.addEventListener("click", (e)=>MyEventHandler(e, "Bubbling"));
	//actually, this will be handler for capturing and target phase
	div3.addEventListener("click", (e)=>MyEventHandler(e, "Capturing"), true);
	//actually, this will be handler for bubbling and target phase
	div3.addEventListener("click", (e)=>MyEventHandler(e, "Bubbling2"));
	//actually, this will be handler for capturing and target phase
	div3.addEventListener("click", (e)=>MyEventHandler(e, "Capturing2"), true);
}       
```

Please note that in (\*\*) we interleaved definition of Capturing and Bubbling event handlers for the same element.    

### 4.3 Screenshot   
Here is the application screenshot.   

![39_41pic.png](Readme/39_41pic.png)

### 4.4 Execution – Our Click   

Now we do our click. Here is a log that is produced:   

![39_43pic.png](Readme/39_43pic.png)

As far as I see (on this version of Chrome), in the Target phase, all events are run in order of Event-Handlers Capturing first, then Bubbling. Handlers are not run in order as defined in (\*\*).   
If you look at the Target Phase (yellow), you will see the order of events “Capturing”, “Capturing2”, “Bubbling”, “Bubbling2” in this execution. What some people on the internet claimed is that the order will be the same as in definition (\*\*), something like “Capturing”, “Bubbling”, “Capturing2”, “Bubbling2”. But, this experiment of mine disproves that claim, at least for this version of Chrome.    

## 5 Conclusion   
In this article, we provided a simple “proof-of-concept” application that shows how event propagation in DOM works.    

## 6 References   
[1] https://javascript.info/    
[2] https://dom.spec.whatwg.org/#eventtarget    



