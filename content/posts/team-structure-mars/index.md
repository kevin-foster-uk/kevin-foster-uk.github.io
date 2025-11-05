
---
title: "How a Flawed Team Responsibility Structure Destroyed a Mars Mission"
draft: false
date: 2025-09-09T18:22:00.000Z
description: "By designing teams for clarity and ownership first, we reduce cognitive load, eliminate fuzzy handoffs, and build in quality from the start."
categories:
  - Software Dev
  - Teams
  - Company Culture
tags:
  - Software Dev
  - Teams
  - Company Culture
---


In 1999, NASA’s Mars Climate Orbiter was minutes from entering orbit around the Red Planet. Instead, it flew too close to the atmosphere and burned up, vanishing without a trace. The cause? A software error.

But not the kind you might think.

The error wasn’t a complex algorithm flaw. It was a unit conversion. One software team at Lockheed Martin calculated thruster force in imperial pound-seconds. Another team at NASA’s JPL, who were guiding the spacecraft, expected the data in metric newton-seconds. The failure to convert between the two caused a navigation error so severe it doomed the $125 million mission.

This wasn’t just an engineering mistake. It was a catastrophic failure of team responsibility structure.

**The Flaw:** Shared Responsibility, No Clear Ownership

The project's structure created a deadly ambiguity. Two brilliant teams worked on different components, but the responsibility for the interface between them was shared and unclear. This "multi-team component" approach meant the data handoff was a fragile process, relying on documentation and human communication, a system guaranteed to fail under pressure. No single team was empowered to own the integrity of the entire thruster data stream.

**The Fix:** Designing Teams for Clear Ownership

The book Team Topologies by Matthew Skelton and Manuel Pais provides the blueprint for preventing this exact disaster. The solution is to design team structures that mandate clear ownership and minimise ambiguous handoffs.

Instead of organizing teams around components (the thrusters vs. the navigation system), you organise them around streams of change and clear interfaces.

1. Form a Stream-Aligned Team with E2E Ownership: A single, cross-functional team should have end-to-end ownership of the core mission, in this case, "Orbital Insertion." This team is empowered with all the skills to manage the navigation and the data it depends on.
2. Define Platforms with API Contracts: The thruster system, provided by another team, shouldn't be a vague component. It should be a platform with a well-defined, immutable API. Imagine the Lockheed team providing a service with a function called getThrustInNewtonSeconds().
3. Clarify Responsibility through Interfaces: The navigation team simply calls that function. The unit of measurement is baked into the function name and return type, making a mix-up impossible for a developer to miss and trivial for a test to catch. The API is the contract, and the platform team owns everything behind it.

**The Bottom Line:** Structure Teams for Clarity

The Mars Climate Orbiter’s fate was sealed not when the code was written, but when the org chart was drawn. The software architecture will always reflect the communication structure of the teams that build it, a principle known as Conway’s Law.

A sound team responsibility structure isn't about organisational charts; it's the bedrock of system reliability. By designing teams for clarity and ownership first, we reduce cognitive load, eliminate fuzzy handoffs, and build in quality from the start.

In software, a flawed team structure causes outages and frustration. In aerospace, it costs $125 million and a mission to Mars.

Let’s learn from the past and build our teams as intentionally as we build our systems.

To learn more about structuring teams for success, visit https://teamtopologies.com/.

