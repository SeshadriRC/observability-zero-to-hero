**Getting Started with Synthetics Monitoring**

Synthetic monitoring proactively simulates real user interactions with your applications before your customers experience issues. Unlike traditional monitoring that waits for problems to occur, synthetic monitoring continuously tests your critical user journeys—like login, checkout, or search—from the outside-in perspective. Elastic Synthetics provides comprehensive monitoring capabilities, from simple uptime checks to complex multi-step user journeys using real browsers.

While logs tell you what happened, metrics show you how your system is performing, and traces reveal how requests flow through your services, synthetic monitoring tells you whether your users can actually accomplish their goals. It bridges the gap between internal technical health and user experience. When your checkout process fails, for instance, synthetic monitoring detects it immediately and exactly as the user would have experienced it. This is true even if your servers are healthy and your metrics look normal. This outside-in perspective provides early warning for issues that other observability data might miss and complements the other signals to paint a complete picture of your system.

**Elastic Synthetics offers two main categories of monitors:**

Lightweight ping monitors (HTTP/HTTPS, TCP, and ICMP) that test basic network connectivity, service availability, and endpoint responses
Browser monitors that execute complete multi-step user journeys using real Chromium browsers.
All monitors can run from Elastic's global managed testing infrastructure across multiple geographic locations or from private locations on your own infrastructure. Results integrate seamlessly with your existing Elastic Stack, allowing you to correlate synthetic test failures with logs, metrics, and APM traces for faster root cause analysis.

**In this track, you will:**

Understand the core components of Elastic Synthetics (lightweight vs browser monitors, locations, scheduling)

Create and schedule lightweight and browser monitors using the Synthetics UI

Learn to write Playwright journeys that simulate real user interactions with your applications

Simulate a service outage and monitor the user experience before, during, and after the outage

⚠️ You are starting with an empty Elastic deployment. If you start over, your progress will be lost.

⏳ The setup will take at least 30 seconds - we are deploying an application that you can troubleshoot now. If you see a loading bar at the bottom of this screen, it may take longer.
