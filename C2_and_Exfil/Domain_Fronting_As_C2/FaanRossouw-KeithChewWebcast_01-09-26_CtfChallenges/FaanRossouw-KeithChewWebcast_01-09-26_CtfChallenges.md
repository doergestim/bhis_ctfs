<a name="_hxuop7blruj4"></a>**Level 1: The Initial Triage**

**Context:** You’ve received an alert about suspicious remote access activity involving encrypted traffic. You have the network logs.

**Question 1:** Which internal IP is most likely under C2 control?

- **Answer:** 10.0.0.4

**Question 2:** What external IP is controlling it over SSH?

- **Answer:** 152.53.209.147

**Question 3:** What SSH client banner indicates automated tooling?

- **Answer:** SSH-2.0-Go

![](Aspose.Words.e520d091-3115-4550-893e-4cc8e36e3741.001.png)

<a name="_8pehi9kawh1o"></a>**Level 2: Traffic Analysis**

**Context:** The suspected C2 channel is active. You now analyze protocol behavior for confirmation.

**Question 4:** Which protocol shows violations in analyzer.log?

- **Answer:** RDP

**Question 5:** Which two other protocols support the C2 hypothesis besides SSH?

- **Answer:** DNS and HTTP

**Question 6:** Which direction is the suspicious SSH session?

- **Answer:** Inbound ( External -> Internal )


