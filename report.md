---
title: "Pacific Tours CCSE-CW1 Report"
author: "2242090"
bibliography: references.bib
toc: true
toc-title: Table of Contents
toc-depth: 3
csl: harvard-imperial-college-london.csl
---

# 1 Introduction

This report documents the design, development, and implementation of a web-based reservation system for the company, Pacific Tours. The project aims to create an online system that enables customers to search, select, and reserve various hotel accommodations and tour packages that are offered by the company.

The main requirements for this system are as follows:

- Customer registration and login
- Searching and booking hotels
- Searching and booking tours
- Searching and booking packages (consisting of hotel + tour)
- Offering discounts based on booked packages
- Managing booking cancellations and modifications
- Generating booking summaries and availability reports for managers

# 2 System design and development approach

To meet the outlined requirements, a software development methodology, specifically agile, was incorporated to iteratively gather requirements, and design considerations to implement and demonstrate credible working progression within set cycles.

## 2.1 Agile software development methodology

The iterative approach to developing software has been highly beneficial to programmers to improve their skills and organisations in estimating the necessary timespan required for certain tasks [@edeki2015]. Its traits of flexibility, a clear-defined scope of requirements, quick adaptability as well and pragmatism to deliverables make it suitable for software companies to survive within evolving landscapes [@brush2022] – whilst maintaining the company's business interests.

Agile programming utilises the idea of sprints, which are iterative repetitions whereby a functionality is taken and developed to produce small new increments. Each sprint follows the typical developmental phases, as seen in more traditional software methodologies such as the waterfall method. These phases include requirements, analysis, design, evolution, and delivery @abrahamsson2017].

## 2.2 Agile application and design considerations

For this specific scenario, however, an agile methodology was adopted with sprints each lasting 2-weeks. This timeframe proved to be adequate for implementing (which arguably took most of the time), demonstrating, and conducting robust testing. Rather than delivering all the requirements at once, it was beneficial to abstract and break the overall scenario down into key components and decompose further if required.

The sprints were broken down into the following as so, assuming that the project took nearly 2-months.

### 2.2.1 Sprint 1

| Tasks                                                                        | Interpretation                                                                                                            |
| ---------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------- |
| Choosing suitable web technologies                                           | ASP.NET C# with both client and server side combined                                                                      |
| Setting up robust development environment                                    | Adequate system resources for high performance, Windows 10 OS, Microsoft SQL Management Studio, stable network connection |
| Familiarising with technologies by developing small Proof of Concepts (POCs) | Experiment with Blazor WebAssembly, ASP.NET default scaffolded classes etc.                                               |
| Deciding the project structure and architecture                              | Single ASP.NET Core Web application with folders for Pages, Services, Exceptions, Models                                  |

### 2.2.2 Sprint 2

| Tasks                              | Interpretation                                                                                  |
| ---------------------------------- | ----------------------------------------------------------------------------------------------- |
| Setup version control system       | Using a private GitHub repository                                                               |
| Configure SQL database settings    | Using Microsoft SQL Management Studio, database context class, Entity Framework Core Migrations |
| Integrate user model               | Using Entity Framework Core Identity, ApplicationUser model                                     |
| Add registration, login and logout | Using ASP.NET scaffolding for default registration, login and logout pages and functionality    |

### 2.2.3 Sprint 3

### 2.2.4 Sprint 4

# 3 System functionality and features

# 4 Cyber security implementation

# 5 Appendices

## 5.1 Link to GitHub repository

[Pacific Tours (pacific-tours-ccse-cw1)](https://github.com/iArcanic/pacific-tours-ccse-cw1)

# 6 References
