# Actor Framework: Abstract Messages How-To
### High-Level Overview
Abstract messages are a method to achieve asynchronous, loosely-coupled messaging upstream from a nested actor to a caller of that actor. Abstract messages are useful because they not only provide a way for nested actors to send messages upstream to callers without tightly coupling the actors, but also allow for writing and sending a payload of data from the nested actor to its caller as well, which is a limitation of self-addressed messages.

The general procedure for creating an abstract message is:
1. Create method in the *receiving* actor (in this case, the parent actor)
2. Create an abstract message in the *sending* actor (the nested actor)
3. Create a child of the newly created abstract message in the *receiving* actor
4. Write the child abstract message into the private member data of the *sending* (nested) actor in the Actor Core.vi of the parent actor before the nested actor is launched
___
### Step-by-Step
Assuming you already have two actors created, one as the parent and the other as the nested actor, follow these steps to create an abstract message:

1. Create method in the receiving actor (in this case, the parent actor)
   1. In your parent actor, create a new message method by right-clicking on the actor’s .lvclass and selecting **New»VI From Dynamic Dispatch Template**.
   
     ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/1.png "Create VI From Dynamic Dispatch Template")
     
   1. Define the functionality of the message method
      * Make sure to not only create the inputs for the method, but **connect them to the connector pane** of the VI. If this is not done, you may have to rescript these messages in the future, which can be messy.
        ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/2.png "Connector Pane")
      * In this example, I have a VI that simply displays a message from the nested actor in a message pop-up.
        ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/3.png "Reference Method Snippet")
2. Create an abstract message in the sending actor (the nested actor)
   1. In the nested actor, right-click on the .lvclass and go to **Actor Framework»Create Abstract Message for Caller**
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/4.png "Create Abstract Message for Caller")
      
   1. In the **New Abstract Message** pop-up, name your message, check the **Use Reference Method** box, and then use the File Dialog button to navigate to the message method you created for the Parent class earlier.
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/5.png "Use Reference Method")
      
      * Doing this allows the Actor Framework auto-scripting to use the predefined method as a template for all of the other VIs and classes it needs to build. This makes it so that your Abstract Message will have the same inputs as your message method, so you don’t have to edit it yourself to make them all consistent.
   1. Once the correct reference method has been selected, click **OK**.
      * Actor Framework will now create all the classes and VIs necessary to flesh out the message for you. Wait until this has completed before clicking on anything else.
   1. Once the VI Scripting has been completed, you should see that the Abstract Messages for Caller virtual folder in your nested actor will now be populated. Expand it to see the newly created abstract message class, with its Send and Read Attributes VI auto-generated.
   
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/6.png "VI Scripting Complete")
      
   1. Before moving on, also notice that the Actor Framework’s VI Scripting has also created a Write accessor for your abstract message and added the abstract message’s class to the private member data of your nested actor.
      * In this example, they are located under **Nested.lvclass** and named **Nested.ctl** and **Write Send to Parent Msg.vi**.
      * These will be used in Step 4 to give the Parent’s message implementation to the nested actor.
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/7.png "Write Accessor")
      
3. Create a child of the newly created abstract message in the receiving actor (the parent actor)
   1. In the parent actor, right-click on the message method VI (in this example, the Send_to_Parent.vi) and select **Actor Framework>>Create Child of Abstract Message**.
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/8.png "Child of Abstract Message")
      
   2. In the navigation window, find the abstract message’s .lvclass file that you just created in the nested actor.
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/9.png "Navigate to Parent Msg.lvclass")
      
   3. Once again, Actor Framework will use automatic VI Scripting to create the message class and VIs for this method, and automatically set up that class to inherit from the nested actor’s Abstract Message.
      * Although this inheritance tree may seem backwards, the reason why we need the parent actor’s message to be a child of the nested actor’s abstract message will become more clear after the next step.
   4. The parent actor’s **Messages for this Actor** virtual folder will now be populated, and you can expand it to see the auto-created .lvclass and Do.vi.
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/10.png "Messages for this Actor")
      
4. Write the child abstract message into the private member data of the sending (nested) actor in the Actor Core.vi of the parent actor before the nested actor is launched.
   1. We now need to pass our message method to the nested actor before it is launched. Typically, nested actors are launched in the Actor Core.vi of its parent actor.
      * If you haven’t already created an override of Actor Core.vi for the parent actor, do so now by right-clicking on your parent actor’s .lvclass and selecting **New»VI for Override…**
        
        ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/11.png "Override Actor Core")
        
      * In the **New Override** pop-up window, select **Actor Core.vi** and click **OK**.
      * The Actor Core.vi override will appear under the parent actor’s .lvclass file
   2. Open the **Actor Core.vi** of your parent actor, open the block diagram, and add the nested actor’s .lvclass cube and the **Launch Nested Actor.vi**.
   2. Add the write accessor of your nested actor’s abstract message to the block diagram and wire the nested actor’s .lvclass wire through the accessor.
   2. Drop the parent actor’s message .lvclass cube onto the block diagram and wire it into the input of the write accessor.
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/12.png "Actor Core.vi")
      
      * Note: The reason you write this message to the member data of the nested actor here in the actor core of the caller is to maintain decoupling. If you did this in the pre-launch init of the nested actor it would create the coupling that we are actually trying to fix with the abstract message
   2. Your abstract message is now complete, you can now access the parent message’s class reference inside the nested actor by unbundling it from the nested actor’s class wire.
      * You can send the abstract message from the nested actor to the parent using the “Send <Abstract Message Name Here>.vi” held in the nested actor’s Abstract Message.lvclass
      
      ![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/13.png "Send Abstract Message from Nested")

___
## GIF Showing Functionality
![alt text](https://github.com/jasonrorr/ActorFrameworkAbstractMessages/blob/master/Support/Images/AbstractMessages.gif "GIF Showing Functionality")

