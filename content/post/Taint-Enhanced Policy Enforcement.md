+++
date = "2016-10-12T21:45:10+08:00"
draft = true
title = "Taint-Enhanced Policy Enforcement"
tags = [ "Android", "Taint", "blog" ]
+++

Taint analysis is a very popular method for program static analysis. In taint analysis, we can mark some variables or memory region as taint, track the propagation of those datas, and find the path from source to sink. And then we can detect some malicious behaviors according to this analysis information. This paper proposed a taint-enhanced policy to detect many kinds of attacks in runtime.
There is a kinds of vulnerabilities that can be trigged by well-designed input from attackers. For instance, SQL injection, XSS and format string vunerabilities. Conventional access control policy(SELinux) cannot detect those attacks, because those operation does not voilate the access rule. Meanwhile, those attacks do not change the control flow. So, this paper proposed the fine-grained taint policies can prevent these attacks.

A previous work tried to monitor each byte of data in memory by taint analysis and detect attacks that overwrite points with untrusted data. These techniques can be used to buffer overflow and related attacks, since benign uses of programs should not involve pointer values from outside. But can not be used to detect the attacks mentioned above. This paper expand its scope to detect much wider range of attacks based on this basic idea.

---
#### Approach ####
This approach contains two parts, taint analysis and policy enforcement.
In taint analysis phase, we should determine the sources of the taint data first. In this paper, they regarded some input functions as the sources, for example read and recv, because the data returned from those functions can be controled by attackers. And marking those datas as tainted. To record the taint information, they assigned one bit for each byte in physical memory. All the taint information bits form a tagmap array.
And then, they will propagate the taint information along with data propagation. For example, assignment statement, arithmetic statement and so on, I will explain in detail later.
Based on the source and taint data propagation, the taint data path can be constructed, and then it's how to detect malicious behavious. In this paper, they just focus on security-critical functions, for example executeQuery and execve. When program invokes these functions, their arguments will first be checked. If the critical argument has been tainted, that is, the this argument is from a unsafe source. Then this invoketion can be seen as a potential malicious behavior. Then, they tried to terminate the program or reject a request.

----
#### Implementation ####
Now, I will explain the implementation of some critical step. Firstly, we have mentioned that they just regarded the input function as the source of the tainted data. For a certain function, the data can be from different place, too. In this example, read function can get data from file or from a networkEndpoint. Only if the fd is a network endpoint, buf will be ragarded as taint data.
As for the representation of taint, I have explained. What I want to note is that the tagmap array much be protacted.it cannot be written by the program or changed by other process.
This is the implementation of taint information propagation. Here I list 3 kind of statements. In assignment statement, the left value will be tainted if the right value is tainted. And the result of arithmetic statement will be tainted if any of the variable in the expression is tainted. As for function call, they will propagate the taint information between caller and callee using a second stack.
Here is the policy to detect the malicious behavior. For example, the first line show the Control-flow hajacking. When program excute jmp instruction, and address is tainted. In this situation, they will regard this is a malicious behavior, and terminate the program. Or for the fourth line, when program invoke a html print function, if its argument contains "script", they will regard this operation as malicious.

-----
#### Measurement ####
According to their experiment, they can detect large scope of attacks. And this method can be used to all kinds of languages. However, for CPU-intensive programs, it will cause large overhead. And if there are implicit information flows, it will result in false negative.

