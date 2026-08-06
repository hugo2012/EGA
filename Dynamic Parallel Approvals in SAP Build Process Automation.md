In many enterprise approval workflows, there is a need for parallel approvals. These approvals are often dynamic—the number of approvers and their identities depend on the business context, making them hard to model using static workflow designs.

SAP Build Process Automation, a low-code solution on SAP Business Technology Platform (SAP BTP), provides tools to design workflows and automate business processes. However, SAP Build Process Automation currently doesn't support truly dynamic parallel branches out-of-the-box.

In this blog post, I’ll walk you through a design workaround to achieve dynamic parallel approvals using a fixed number of branches corresponding to the maximum number of approvers that can be expected at a time, conditional logic, and structured input. This is particularly useful when the maximum number of parallel approvers is known in advance.

 

The Use Case
Let’s say you are building an approval process for a capital waiver request or a purchase request, where, depending on the request amount, up to six approvers may be involved. You want:

Approvals to happen in parallel
The number of approvers to be dynamic
No parallel branch should be triggered if there is no approver in that position
The email or user ID of each approver to be determined at runtime
Challenges in SAP Build Process Automation
SAP Build Process Automation currently allows parallel steps by modeling parallel branches using the parallel branch element. However, there are some limitations. One, dynamic branching is not supported. There’s no built-in loop control that allows iterating through users dynamically within the visual workflow builder. You also need to predefine how many parallel branches will be possible.

 

Solution Overview
We define six parallel branches up front in the process model. Each branch corresponds to a potential approver (e.g., approver1, approver2... approver6).

To manage this dynamically, you can follow these steps:

Create six email fields in the process input context (email1, email2, ..., email6).
In each branch, use a condition step to check whether the corresponding email field is populated.
If the condition passes, the branch proceeds to an approval task assigned to that user.
If the condition fails, the branch ends immediately (is skipped).
Let’s dive deeper into the exact steps

Step 1: Design the Input Context
In this step, we’ll define the process input context with six optional approver email fields:

{
"approvers": {
   "email1": "string",
   "email2": "string",
   "email3": "string",
   "email4": "string",
   "email5": "string",
   "email6": "string"
}
}

Step 2: Add a Parallel Branch with Six Paths
In the workflow editor, drag the parallel branch element into your process. Then, add six branches (one for each potential approver).

 

Adding a Parallel Branch with Six Steps

Step 3: Add a Condition at the Start of Each Branch
In this step, for each branch, insert a condition step to check if the email field is filled:

 

Pseudocode:

Branch 1 – Condition:
${context.approvers.email1} != null && ${context.approvers.email1} !== ""

Branch 2 – Condition:
${context.approvers.email2} != null && ${context.approvers.email2} !== ""


(... and so on until email6).

 

Adding a Condition at the Start of Each Branch

 

Adding a Condition at the Start of Each Branch

Step 4: Add an Action-Like Trigger Email for Each Valid Branch
Below is what this would look like in SAP Build Process Automation.

 

Adding an Action

Step 5: Wait for All Parallel Approvals to Complete
Once the parallel branches finish, there are a couple of optional things you can do. You can aggregate approval responses, log outcomes, or trigger escalation logic if any approval fails.

 

Benefits and Limitations of This Design
This workaround brings about five key benefits:

It supports dynamic execution: The number of actual approvers can vary.
It’s easy to manage with straightforward, condition-based control.
It’s reusable by working across different process types.
It’s safe because it prevents null assignments or errors.
It’s SAP Build Process Automation compatible by using only standard SAP Build Process Automation components.
However, there are three limitations to keep in mind:

The maximum number of branches is fixed.
Each branch needs to be configured manually.
It doesn’t scale well if the number of dynamic approvers changes frequently.
Future Enhancements
In the future, a number of enhancements could be implemented. To start, steps could be allowed to be dynamically instantiated (loop-based or array-based branching). Provider approval tools could have built-in multi-approval handling functionality. And custom script task loops with async-await features could be introduced.

 

Conclusion
While SAP Build Process Automation does not natively support dynamic parallel branching, with a bit of clever structuring and conditional design, we can still simulate dynamic parallel approvals efficiently.

This fixed-branch dynamic execution approach offers a scalable and safe method to model real-world approval scenarios—balancing between SAP Build Process Automation's current capabilities and enterprise workflow needs.

If you're designing workflows where approver count varies per transaction but remains within a predictable range, this approach can be a reliable template.
