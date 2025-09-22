
---
title: "Taming the Beast: The Two Kinds of Complexity in Software"
draft: false
date: 2025-09-22T11:00:00.000Z
description: 'The path to clarity is a habit of questioning. For every piece of complexity you encounter, ask, "Is this essential to the problem, or did we create it?" Actively seek out misfit technologies and simplify over-engineered solutions. By focusing on the problem at hand and choosing tools that truly fit, we can shed the weight of unnecessary complexity and build what matters more effectively.'
categories:
  - Complexity
  - Software Dev
tags:
  - Complexity
  - Software Dev
---

Every software developer knows the feeling of being overwhelmed by complexity. It comes from every direction: user requirements, technology choices, deployment processes, and team dynamics. But what if we could categorise this complexity? Understanding the difference between what is essential and what is self-imposed is the key to building simpler, more robust systems.

First, there is necessary complexity. This is inherent to the problem you are solving. It is the natural result of intricate business rules, real-world constraints, and the core functionality users demand. You cannot remove this complexity without changing the product's fundamental purpose. For example, writing an algorithm to find the most efficient delivery route for hundreds of packages is inherently complex. Building a system that complies with strict financial regulations carries necessary complexity. This is the price of solving a meaningful problem.

Then, there is unnecessary complexity. This is the complexity we introduce. It often stems from good intentions, like over-engineering a solution for hypothetical future needs. Imagine building a microservices architecture because that's “how to handle scale” without properly considering the alternatives and trade offs. The complexity of service discovery, inter-service communication, and distributed data may be entirely unnecessary.

However, unnecessary complexity can also come from a surprising place, under-engineering. This happens when we choose a technology simply because it is popular or familiar, even when it's a poor fit. For instance, using a relational database with rigid schemas that forces you to write complex and fragile mapping code. The right tool, like a document database, would make the solution elegantly simple. Similarly, using a heavy, web framework to build a mostly static website adding layers of configuration and boot-up time without any benefit.

The hardest part is telling them apart. The line is blurry and depends entirely on context.

The path to clarity is a habit of questioning. For every piece of complexity you encounter, ask, "Is this essential to the problem, or did we create it?" Actively seek out misfit technologies and simplify over-engineered solutions. By focusing on the problem at hand and choosing tools that truly fit, we can shed the weight of unnecessary complexity and build what matters more effectively.
