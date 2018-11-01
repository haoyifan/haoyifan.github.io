---
layout: post
title: A Tiny Survey of Memory Power Consumption
category: posts
---

## A Tiny Survey of Memory Power Consumption

##### 4/15/2018

---

Recently I'm working on a research project that needs to accurately estimate power consumption of various convolutional neural network kernels on mobile devices. A senior PhD student in my group sent me [PowerTutor](http://robertdick.org/publications/zhang10oct.pdf) developed by Zhang et al (surprisingly one of the authors is Prof.Robert Dick, who I worked with when TAing EECS373 at the University of Michigan.). The essence of the paper is a model that estimates the power consumption of cellphones purely based on battery sensors output. However, the paper does not address the power model of memory. Both my labmate and I were curious why memory is not included in the model. Is it because memory power is negligible compared with components considered in the paper? Or is it because the memory is integrated into the Qualcomm SoC, and is therefore hard to model independently? Keeping these questions in mind, I started doing some research on the characteristics of memory power consumption. 

__Side note__: Besides excluding memory in the sphere of modeling, PowerTutor also does not explain how the application is able to estimate per-process power consumption. PowerTutor application is able to derive per-process power consumption, which is kind of like the Android built-in "battery" option in settings. Besides, the portion of the power consumption between processes sometimes do not match up between PowerTutor and the built-in "battery" option.

This post serves as a place to keep my discoveries in case I need them when writing related works.

--- 

#### Related Works

Shortly after raising those questions, my collaborators sent me another interesting article [On Understanding the Energy Consumption of ARM-based Multicore Servers](https://www.semanticscholar.org/paper/On-understanding-the-energy-consumption-of-servers-Tudor-Teo/5f7264e24101ac4d42d2ef9cedd5eae8e7512eec) written by Marius et al. The main contribution of the work is to evaluate running server-class workload on wimpy cores(less capable cores). Compared with traditional processors targeting at cloud computing(Intel x64, for example) platform, low power processors (ARM series, for example) might be able to run the same type of workload more efficiently when the die area for computation logic is the same. The paper also discusses a novel formula to model off-chip power consumption (memory, I/O). A major claim of the literature is that balancing the capabilities of components in the computing system will benefit the runtime as well as power consumption of the overall system. Taking the example of memory bandwidth: a strong processor will not make the runtime shorter when the workload is memory bound. With higher memory bandwidth as well as memory parallelism (higher number of outstanding memory requests), aggregate runtime would be shorter, and thus decreasing the total system power consumption. However, the paper does not give a concrete digit on the power consumption of memory compared with CPU.

After some googling, I obtained a copy of [Beyond CPU: Considering Memory Power Consumption of Software](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=2&cad=rja&uact=8&ved=0ahUKEwjAqpb8673aAhXM7IMKHfu7BXgQFggsMAE&url=https%3A%2F%2Fhal.archives-ouvertes.fr%2Fhal-01314070%2Fdocument&usg=AOvVaw2WvcrsJNfIuj1icb88YAcI) written by Acar et al. Personally I do not think this paper is well written. With some typos, as well as lacking further analysis and explanation, this paper did not give me too much insights except for a rough quantative comparison between CPU power and memory power. The paper also does not explain why the Java compiler cannot apply some trivial optimization on the experiment workloads. The main claim in the paper is that memory power consumption is negligible compared with CPU power consumption. The graphs in the evaluation section depict the power consumption statistics overtime. Indeed, memory power consumption tends to be more stable than CPU power consumption in general. However, CPU power consumption could be only twice as large as memory power consumption in some rare points. This looks especially suspicious to me, as the order of magnitude is almost the same between CPU power and memory power. Since the evaluation workloads does not convince me its credibitily, I continued searching on related papers.

I was then lucky enough to find [An Analysis of Power Consumption in a Smartphone](https://www.google.com/url?sa=t&rct=j&q=&esrc=s&source=web&cd=1&cad=rja&uact=8&ved=0ahUKEwjyoKWp7r3aAhWm7IMKHXAFDdwQFggnMAA&url=https%3A%2F%2Fwww.usenix.org%2Fevent%2Fatc10%2Ftech%2Ffull_papers%2FCarroll.pdf&usg=AOvVaw1Ts7B0bHX65PBjEIUuXuSB) written by Carrol et al. The paper characterize power consumption of individual components in smartphones, which is exactly what I'm looking for. Besides, the related works section helps me find many other useful references in my later research. The evalution is based on SPEC CPU2000 micro benchmark. The paper claims that memory power consumption is negligible in practice, which coincides with the conclusion of Acar et al. However, _mcf_ benchmark gives surprising results: __in extreme cases, memory power consumption can actually be greater than that of CPU power, albeit a small margin__. This point somewhat invalidates the approach made by PowerTutor, which does not include memory power model. In related works of the paper, it cites paper [Analysis of Dynamic Power Management on Multi-Core Processors](https://dl.acm.org/citation.cfm?id=1375575) written by Bircher et al. Evalutions in this paper also confirms that _mcf_ will make memory power consumption higher than CPU power consumption.

#### Thoughts

It should be pretty clear that even if memory power consumption is one magnitude smaller than CPU power consumption in normal cases, memory power could get really large, even exceeding that of CPU in some rare cases. Here're my questions and what I learned after this 4 hour research in google:
- All the previous papers use scientific computing workloads or commercial workload to evaluate the system. What would happen to the system power consumption if the workload is instead a CNN/RNN/DNN kernel? Is the memory power going to be consistently smaller/larger than CPU power? What are the characteristics of these neural network workload that would make the system behave this way?
- SoC does not necessarily contain a on-chip memory (as what is shown from Carrol et al.) What are the implications of neural network kernels running in SoC? I believe there should be already tons of works addressing this issue.

#### Reference

- Zhang, Lide et al. “Accurate online power estimation and automatic battery behavior based power model generation for smartphones.” 2010 IEEE/ACM/IFIP International Conference on Hardware/Software Codesign and System Synthesis (CODES+ISSS) (2010): 105-114.
- Tudor, Bogdan Marius and Yong Meng Teo. “On understanding the energy consumption of ARM-based multicore servers.” SIGMETRICS (2013).
- Bircher, William Lloyd and Lizy Kurian John. “Analysis of dynamic power management on multi-core processors.” ICS (2008).
- Carroll, Aaron and Gernot Heiser. “An Analysis of Power Consumption in a Smartphone.” USENIX Annual Technical Conference (2010).
- Acar, Hayri et al. “Beyond CPU: Considering memory power consumption of software.” 2016 5th International Conference on Smart Cities and Green ICT Systems (SMARTGREENS) (2016): 1-8.
