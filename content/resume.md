---
title: "Aaryn Smith's Resume"
type: "resume"
description: "Aaryn Smith's Resume"

primaryColor: "#0c7a17"
textPrimaryColor: "#1c1c1c"

profile:
  enable: true
  name: "Aaryn Smith"
  tagline: "Software Engineer"
  # avatar: "img/Gopher.png"

sumsect:
  enable: true
  text: "Experienced developer with a strong background in backend and frontend development, systems management, and network architecture. My objective is to leverage my skills and expertise to contribute to a dynamic software development team. I am eager to utilize my technical proficiency and industry knowledge to drive innovation, and deliver high-quality software solutions. "

contact:
  enable: true
  location: "DFW, Texas"
  list:
    - { 
        icon: "fa fa-phone",
        url: "tel:6825039982", 
        text: "(682) 503-9982" 
      }
    - {
        icon: "fa fa-envelope",
        url: "mailto:aaryn.smith<at>gmail.com",
        text: "aaryn.smith<at>gmail.com",
      }
    - { icon: "fa fa-globe", url: /, text: "aarynsmith.com" }

education:
  enable: true
  list:
    - degree: Master's of Science Degree in Information Technology - Application Development
      gpa: 3.9
      university: "Southern New Hampshire University, Manchester, New Hampshire"
      dates: "2023"
    - degree: Bachelor's of Science Degree in Network and Communications Management
      university: "DeVry University, Irving, Texas"
      dates: "2009"

experience:
  enable: true
  list:
    - title: Full Stack Web Developer
      dates: 2023-Present
      company: Creare Development
      items:
        - details: Design and maintain websites using frameworks such as React, SvelteKit, and Vue to ensure visually appealing and accessible user interfaces
        - details: Implement interactive features using JavaScript and TypeScript to create dynamic and engaging web pages
        - details: Manage and design databases in both SQL and NoSQL environments
        - details: Integrate and design APIs, allowing seamless data exchange
        - details: Writing and executing tests to ensure the code's functionality and reliability
        - details: Staying updated with the latest trends, technologies, and best practices in web development
        - details: Continuously improving skills and adapting to evolving industry standards
    - title: Developer/Network Administrator/Systems Analyst
      dates: 2009-2018
      company: HFSI
      items:
        - details: Developed end-user programs and conversion programs
        - details: Developed applications consuming data from many file formats and standards
        - details: Optimized and developed workflows for several business processes
        - details: Managed multiple Microsoft SQL databases and PostgreSQL databases
        - details: Designed and optimized database queries for analysis and maintenance
        - details: Managed and maintained several medium-sized business networks
        - details: Implemented and maintained a VMWare virtual infrastructure environment
        - details: Provided support and service to several small to medium-sized banks

        # - details: "Developed end user programs and conversion programs, dealing with multiple file formats and standards"
        # - details: "Optimized and developed workflows for several business processes"
        # - details: "Managed and maintained a medium-sized business network"
        # - details: "Managed several Microsoft Windows servers including Active Directory and Microsoft SQL Server"
        # - details: "Managed several Cisco network devices including ASA, FIREpower IPS, and Catalyst switches"
        # - details: "Managed multiple Microsoft SQL databases and PostgreSQL databases"
        # - details: "Implemented and maintained a VMWare virtual infrastructure environment"
        # - details: "Provided support and service to several small- to medium-sized banks"
    - title: Network Technician
      dates: 2006 - 2009
      company: The Lance Brown Agency
      items:
        - details: Designed, installed, and maintained a working office network
        - details: Created and maintained a website for the insurance agency

        # - details: "Designed, installed, and maintained a working office network"
        # - details: "Created and maintained a website for the insurance agency"
        # - details: "Designed, packaged, and distributed marketing materials"
    # - title: "Closed Circuit Television Operator, Contractor"
    #   dates: 2007 - 2008
    #   company: Electronic Systems Services
    #   items:
    #     - details: "Acquired and retained high-risk security clearance"
    #     - details: "Monitored closed-circuit security systems"
    #     - details: "Reported security violations to Bureau of Engraving and Printing"

projects:
  enable: true
  list:
    - title: ztDNS
      meta: Open Source
      tagline: >-
        Created a custom DNS server for ZeroTier VPN Service that integrates
        with the ZeroTier API to provide DNS entries for other devices on
        the network.
    - title: RoundUp Program
      meta: HFSI
      tagline: >-
        Developed rewards program for a bank. Developed program, user
        interaction, and workflow for a program that read customer purchase
        amounts from real time activity file, rounded to the nearest dollar,
        and applied credit to customer savings accounts.
    - title: COBOL Conversion
      meta: HFSI
      tagline: >-
        Modernized COBOL programs to work in updated environments. Recreated
        workflows for several 16-bit COBOL programs in modern languages to
        work in 32- and 64-bit environments. Programs were converted to Go,
        C++, or Python.
    - title: Email Migration
      meta: HFSI
      tagline: >-
        Implemented Google-Hosted Email. Transitioned mailboxes to Google G
        Suite hosted email. Trained employees on new email features and G
        Suite workflows.
    - title: VDI Investigation
      meta: HFSI
      tagline: >-
        Prototyped and tested VDI solutions for use in a SAAS environment. 
        Evaluated cost versus return on investment for VDI solutions from Amazon 
        and VMware to determine viability for offering a VDI product to customers.

information:
  enable: false
  list:
    # - title: Certifications
    #   items:
    #     - details: 2020 FreeCodeCamp - Full Stack
    #     - details: 2020 FreeCodeCamp - Information Security
    #     - details: 2020 FreeCodeCamp - Front End Libraries
    #     - details: 2020 FreeCodeCamp - APIs and Microservices
    #     - details: 2020 FreeCodeCamp - Responsive Web Design
    #     - details: 2020 FreeCodeCamp - Data Visualization
    #     - details: 2020 FreeCodeCamp - JavaScript Algorithms and Data Structures
    #     - details: 2017 Global Knowledge - Cisco ASA with FirePower Services v2.1
    #     - details: 2013 Global Knowledge - VPN 2.0 - Deploying Cisco ASA VPN Solutions

certificates:
  enable: true
  list:
    - certificate: Responsive Web Design
      organization: FreeCodeCamp
      dates: "2020"
    - certificate: JavaScript Algorithms and Data Structures
      organization: FreeCodeCamp
      dates: "2020"
    - certificate: Cisco ASA with FirePower Services v2.1
      organization: Global Knowledge
      dates: "2017"
    - certificate: VPN 2.0 - Deploying Cisco ASA VPN Solutions
      organization: Global Knowledge
      dates: "2013"


awards:
  enable: true
  list:
    - name: Honor Roll
      body: Southern New Hampshire University
      date: "2022 - 2023 (8 times)"

skills:
  enable: true
  list:
    - title: Technical
      items:
        - details: Javascript/Typescript/Node.js
        - details: React/Redux/JQuery/Sass
        - details: HTML/CSS/Bootstrap
        - details: MongoDB/Mongoose
        - details: Socket.io/Mocha/Chai
        - details: Docker/Docker-Compose
        - details: PostgreSQL/MySQL/Microsoft SQL
        - details: Go
        - details: Git and Gitlab/Github CI/CD
        - details: Bash/Windows Batch/PowerShell
        - details: C/C++/Embedded Systems
        - details: Python/Ruby/Perl
        - details: Design and implement database structures
        - details: Lead and deliver complex software systems
    - title: Network Management
      items:
        - details: Cisco ASA/FirePower IPS/Catalyst Switches
        - details: VMWare VSphere/ESX
        - details: Microsoft Windows/Linux Servers
        - details: PFSense/Sophos/SonicWall Firewalls
        - details: Avaya/Alcatel/Asterisk phone systems
    - title: Professional Skills
      items:
        - details: Effective communication
        - details: Team player
        - details: Strong problem solver

languages:
  enable: false
  list:
    - name: English
      level: Native

interests:
  enable: false
  list:
    - name: Climbing
    - name: Snowboarding
    - name: Photography
    - name: Travelling

social:
  enable: true
  list:
    - icon: fab fa-github
      url: //github.com/AarynSmith
    - icon: fab fa-free-code-camp
      url: //www.freecodecamp.org/aarynsmith
    - icon: fa fa-envelope
      url: "mailto:aaryn.smith<at>gmail.com"

footer:
  copyright: "Aaryn Smith"
---
