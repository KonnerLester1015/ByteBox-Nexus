---
title: "Subnet Calculator"
sidebar:
  exclude: true
---

## Introduction

This document provides background information on the purpose and usage of the **Subnet Calculator**.  
I created this tool while taking a *Network Fundamentals* course at **Purdue University**. The course included several subnetting assignments that required performing binary calculations using a **host IP address**, a **major network mask**, and a **subnet mask**.

Using these inputs, I had to perform bitwise operations to determine several key values, as shown in **Table 1**.

**Table 1 – Example Subnet Sheet**

| Variable | Example Value |
|-----------|----------------|
| Host IP Address (Provided) | 219.2.3.201 |
| Major Network Mask (Provided) | 255.255.0.0 |
| Major Network Address | 219.2.0.0 |
| Major Network Broadcast Address | 219.2.255.255 |
| CIDR Notation Major Network | 219.2.0.0/16 |
| Number of Host Bits | 16 |
| Number of Hosts | 65,534 |
|  |  |
| Subnet Mask (Provided) | 255.255.255.192 |
| Number of Subnet Bits | 10 |
| Number of Usable Subnets | 1,023 |
| Number of Host Bits per Subnet | 6 |
| Number of Usable Hosts per Subnet | 62 |
|  |  |
| Subnet Address for this Host | 219.2.3.192 |
| First Host IP Address for this Subnet | 219.2.3.193 |
| Last Host IP Address for this Subnet | 219.2.3.254 |
| Broadcast Address for this Subnet | 219.2.3.255 |
|  |  |
| Network Address of the 5th Usable Subnet | 219.2.1.0 |
| Broadcast Address of the 5th Usable Subnet | 219.2.1.63 |

---

## Technology Stack

The calculator’s **user interface** is built using **HTML** and styled with **CSS**, while all subnetting calculations are handled using **Python**.

This project leverages **PyScript** to run Python directly in the browser, allowing the `main.py` script to interact with HTML elements **without requiring a backend server**.

---

## Features

- **Interactive Web Interface:** Intuitive UI for entering network parameters.  
- **Comprehensive Calculations:** Computes all major network and subnet values, as shown in Table 1.  
- **Flexible Options:** Allows users to include or exclude “all 0’s” and “all 1’s” subnets, affecting usable subnet counts.  
- **Nth Subnet Calculation:** Determines the network and broadcast address for any *nth* usable subnet.  
- **Terminal Version:** A standalone Python script (`SubnetCalculator - Terminal.py`) offers equivalent functionality via the command line.

---

## Project Files

| File | Description |
|------|--------------|
| `index.html` | Main HTML file for the user interface. |
| `main.py` | Python script containing subnetting logic (executed by PyScript). |
| `styles.css` | Stylesheet for the web interface. |
| `SubnetCalculator - Terminal.py` | Command-line version of the calculator. |
| `pyscript.toml` | Configuration file for PyScript. |

{{< callout type="info" icon="github" >}}
  To access the project source code, visit the GitHub repository: [Subnet-Calculator](https://github.com/KonnerLester1015/Subnet-Calculator)
{{< /callout >}}

---

## Usage

You can use the Subnet Calculator directly at: [**konnerlester1015.github.io/Subnet-Calculator**](https://konnerlester1015.github.io/Subnet-Calculator/)

1. Enter the required network parameters.  
2. Click **Submit** to view calculated results.

**Input Example:**  
![Subnet Calculator Input](/SubnetCalculatorInput.png)

**Output Example:**  
![Subnet Calculator Output](/SubnetCalculatorOutput.png)
