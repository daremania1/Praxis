Project Status Summary

Our project's goal was to build an autonomous SOC agent to detect threats and reduce analyst "alert fatigue." Our research was a two-part journey: we first tested a novel idea that failed, and then used that failure to build a much better system that succeeded.

Experiment 1: The Reinforcement Learning (RL) Agent (The "Instructive Failure")
•	What We Did: We first built a Deep Q-Network (DQN), a type of RL agent. The idea was that it could learn from its mistakes.
•	The Result: Our experiment showed this agent was not viable. While it learned to find attacks, it had a catastrophically high False Positive Rate (FPR). Its confusion matrix showed it incorrectly flagged thousands of Benign (normal) samples as attacks.
•	The Conclusion: This agent worsens alert fatigue. It cries wolf constantly. We proved this approach was the wrong tool for the job.

Experiment 2: The Random Forest (RF) Model (The "Successful Detector")
Based on our first failure, we pivoted. We decided to build and test a traditional (but powerful) RandomForestClassifier to see if it could be a reliable "Detector."
1. Data & EDA (What We Found)
We loaded and cleaned all our .parquet files from Google Drive. Our analysis of the data showed two key things:
•	The Data is Highly Imbalanced: The dataset is "attack-heavy." Benign traffic is only 22.68% of the data. Some attacks (like DrDoS_NTP) are very common (28.14%), while others (like WebDDoS) are incredibly rare (0.01%).
•	Attacks Have "Fingerprints": We proved that different attacks look completely different. A Syn attack (tiny packets, low rate) has a different numerical "fingerprint" than a DrDoS_LDAP attack (huge packets, massive rate). This means a model can learn the patterns.

2. Training & Evaluation
We trained our RF model and then tested it on data it had never seen before. The results were a massive success:
•	It Solved Alert Fatigue: The model achieved a perfect 1.00 F1-score for Benign traffic. This is the most important finding. It means our model can reliably ignore normal traffic and stops the false alarms our RL agent created.
•	It Was Accurate (Mostly): It was also perfect (1.00 F1) on common attacks like DrDoS_NTP and TFTP, and near-perfect (0.99 F1) on Syn.
•	Its Weakness: It failed on the rare attacks (like WebDDoS), proving our model's performance is tied to the imbalance we found in the EDA.

3. Interpretation (Why It Worked)
We then investigated why the model was so good. We used Feature Importance to ask the model what it was looking at.
•	The Answer: The model's "brain" overwhelmingly focuses on packet size features (like Packet Length Min, Avg Fwd Segment Size, etc.). It successfully learned the "fingerprints" we found in our EDA.

Final System: The Agentic AI (The "Full Solution")
With a proven, successful "Detector" (our RF model), we built our final system. This is the Autonomous "Detective" Agent.
•	Assembly: We used LangChain to assemble our final agent.
•	The "Brain": We used a Gemini LLM (via our API key) to act as the reasoning engine.
•	The "Tools": We gave the "brain" two custom tools:
1.	rf_detector_tool (The "Guard"): Our proven, high-accuracy Random Forest model.
2.	mitre_retriever_tool (The "Handbook"): An intelligence database of the MITRE ATT&CK framework.
•	The Demo: We built a Gradio UI to demonstrate the agent's full, end-to-end workflow:
Detect (RF) -------- Enrich (MITRE) ----------- Reason & Report (LLM).


Status of Our Original Research Hypotheses
This is how our research answers our original questions.
•	Hypothesis 1 (Effectiveness): "The deployment of Agentic AI for autonomous threat hunting improves the threat detection rate (TDR)... compared to Random Forest Model"
o	Status: REJECTED. Our work proves the opposite: the Random Forest is the superior detector. Our final agent uses the RF model, it doesn't try to beat it.

•	Hypothesis 2 (FPR Reduction): "The Agentic AI system reduces the False Positive Rate... by at least 25% than the Random Forest Baseline"
o	Status: REJECTED. This is our most important finding. Our first "Agentic AI" (the RL agent) had a terrible FPR. The Random Forest baseline had a perfect FPR (1.00 F1 on Benign). Our research proves we must use the baseline to achieve this goal.

•	Hypothesis 3 (MITRE Integration): "Integrating the MITRE ATT&CK framework... will reduce the mean time to detect (MTTD) simulated advanced persistent threats (APTS)..."
o	Status: READY FOR TESTING (Future Work). We have not run this experiment. However, we have successfully built the agent that is needed to test this hypothesis.

•	Hypothesis 4 (Playbook Precision): "Agentic AI systems that execute hunts using open-source Threat Hunter Playbook procedures will achieve at least 12% higher detection precision..."
o	Status: NOT VALIDATED (Future Work). This is a future research step. We would first need to add a "Playbook" as a third tool to our agent.

•	Hypothesis 5 (Generalization): "Agentic AI systems trained in CyberBattleSim... will generalize to detect 20% more novel attack techniques in real-world datasets..."
o	Status: NOT VALIDATED (Future Work). We have not done any simulation/built on the CyberBattleSim
