# Saving-Group-Tracker
Savings Circle 

## Group Members

- Bame Melissa Balole 202503755
- Brian Thabang Ramhago 202400664
- Keabetswe Tsholofelo Ramakgala 202505789
- Aone Ropane 202402953
- Tlotlo Kobedi 202202424

## Chosen Domain

**Finance & Savings – Motshelo Management** – Digital tracker for student rotating savings groups.

## Concept Note

Objective
The Savings Group Tracker is designed to assist community members in managing a savings group effectively. The application allows users to track contributions, manage budgets, and generate financial reports.

Background
Savings groups play a crucial role in promoting financial stability in communities across Botswana and Southern Africa. This application aims to streamline the process of tracking contributions and managing group finances, making it easier for members to stay accountable and organized.

**Key Features**
1. Member Management.. Add and maintain member details.
2. Contribution Tracking:Log individual contributions to the savings pool.
3. Budget Management. Set savings goals and monitor progress against them.
4. Reporting.Generate reports on contributions and budget status to promote transparency.

**The problem:** Currently tracked in notebooks which have:
- Calculation errors
- Lost or damaged books
- Difficult to see who hasn't paid
- No payment history

**Our solution:** A Java console app that:
- Tracks member contributions
- Shows who has paid vs who owes
- Calculates the pot (total savings)
- Shows whose turn is next to receive
- Generates payment reports

**Local flavour:** Community leaders and members involved in informal savings groups looking for a simple tool to help manage their finances

*Submitted for CSI142 Milestone 1 – 10 April 2026*

Finance-Tracker/
│
├── src/
│   ├── Main.java
│   ├── model/
│   │   ├── User.java
│   │   ├── Account.java
│   │   ├── SavingsAccount.java
│   │   └── Transaction.java
│   ├── service/
│   │   ├── GroupManager.java
│   │   └── ReportGenerator.java
│   └── utils/
│       └── FileHandler.java
│
├── data/
│   └── savings_data.txt
│
├── .gitignore
├── README.md

CLASS LIST
==========

1. Main
2. MemberManager
3. PaymentTracker
4. ReportGenerator
5. User
6. Account
7. SavingsAccount
8. Transaction
9. FileHandler

TOTAL: 9 CLASSES

import java.util.Scanner;

/**
 * Finance Tracker - Community Savings Group
 * CSI142 Project
 * ONLY menu loop is here - all logic is in separate classes
 */
public class Main {
    
    // Global scanner for input
    private static Scanner scanner = new Scanner(System.in);
    
    // Create instances of manager classes
    private static MemberManager memberManager = new MemberManager();
    private static PaymentTracker paymentTracker = new PaymentTracker();
    private static ReportGenerator reportGenerator = new ReportGenerator();
    
    public static void main(String[] args) {
        
        System.out.println("=====================================");
        System.out.println("       FINANCE TRACKER               ");
        System.out.println("    Community Savings Group          ");
        System.out.println("=====================================");
        
        // Load saved data
        memberManager.loadData();
        
        int choice;
        
        do {
            // Display menu
            displayMenu();
            
            // Get user choice
            System.out.print("Choice: ");
            choice = scanner.nextInt();
            scanner.nextLine(); // Clear buffer
            
            // Handle choice using switch - ONLY MENU LOOP HERE
            switch (choice) {
                case 1:
                    addMember();
                    break;
                case 2:
                    recordContribution();
                    break;
                case 3:
                    viewPaymentStatus();
                    break;
                case 4:
                    viewTotalPot();
                    break;
                case 5:
                    showNextReceiver();
                    break;
                case 6:
                    generateReport();
                    break;
                case 7:
                    setBudgetGoal();
                    break;
                case 8:
                    System.out.println("\nSaving data...");
                    memberManager.saveData();
                    System.out.println("Goodbye!");
                    break;
                default:
                    System.out.println("Invalid choice. Please enter 1-8.");
            }
            
        } while (choice != 8);
        
        scanner.close();
    }
    
    /**
     * Display the main menu
     */
    private static void displayMenu() {
        System.out.println("\n+---------------------------------+");
        System.out.println("|           MAIN MENU              |");
        System.out.println("+---------------------------------+");
        System.out.println("| 1) Add Member                   |");
        System.out.println("| 2) Record Contribution          |");
        System.out.println("| 3) View Payment Status          |");
        System.out.println("| 4) View Total Pot               |");
        System.out.println("| 5) Show Next Receiver           |");
        System.out.println("| 6) Generate Report              |");
        System.out.println("| 7) Set Budget Goal              |");
        System.out.println("| 8) Save & Exit                  |");
        System.out.println("+---------------------------------+");
    }
    
    /**
     * Case 1: Add a new member
     */
    private static void addMember() {
        System.out.println("\n--- ADD NEW MEMBER ---");
        
        System.out.print("Enter member name: ");
        String name = scanner.nextLine();
        
        System.out.print("Enter student ID: ");
        String id = scanner.nextLine();
        
        memberManager.addMember(name, id);
        
        System.out.println("Member " + name + " added successfully!");
        System.out.println("Total members: " + memberManager.getMemberCount());
    }
    
    /**
     * Case 2: Record a contribution
     */
    private static void recordContribution() {
        System.out.println("\n--- RECORD CONTRIBUTION ---");
        
        if (memberManager.getMemberCount() == 0) {
            System.out.println("No members in the group. Add a member first.");
            return;
        }
        
        System.out.print("Enter member name: ");
        String name = scanner.nextLine();
        
        if (!memberManager.memberExists(name)) {
            System.out.println("Member not found!");
            return;
        }
        
        System.out.print("Enter amount (P): ");
        double amount = scanner.nextDouble();
        scanner.nextLine(); // Clear buffer
        
        if (amount <= 0) {
            System.out.println("Amount must be positive!");
            return;
        }
        
        paymentTracker.recordPayment(name, amount);
        
        System.out.printf("Recorded", amount, name);
        System.out.printf("Total paid", name, paymentTracker.getMemberPaid(name));
    }
    
    /**
     * Case 3: View payment status
     */
    private static void viewPaymentStatus() {
        System.out.println("\n--- PAYMENT STATUS ---");
        
        if (memberManager.getMemberCount() == 0) {
            System.out.println("No members in the group.");
            return;
        }
        
        reportGenerator.printPaymentStatus(
            memberManager.getAllMembers(),
            paymentTracker.getAllPayments(),
            paymentTracker.getExpectedAmount()
        );
    }
    
    /**
     * Case 4: View total pot
     */
    private static void viewTotalPot() {
        System.out.println("\n--- TOTAL POT ---");
        double total = paymentTracker.getTotalPot();
        System.out.printf("Current total savings:, total);
    }
    
    /**
     * Case 5: Show next receiver
     */
    private static void showNextReceiver() {
        System.out.println("\n--- NEXT RECEIVER ---");
        
        if (memberManager.getMemberCount() == 0) {
            System.out.println("No members in the group.");
            return;
        }
        
        String nextReceiver = memberManager.getNextReceiver();
        System.out.println("Next to receive the pot: " + nextReceiver);
    }
    
    /**
     * Case 6: Generate full report
     */
    private static void generateReport() {
        System.out.println("\n+==================================================+");
        System.out.println("|                 FULL FINANCIAL REPORT              |");
        System.out.println("+==================================================+");
        
        if (memberManager.getMemberCount() == 0) {
            System.out.println("| No members in the group.                            |");
        } else {
            reportGenerator.printFullReport(
                memberManager.getAllMembers(),
                paymentTracker.getAllPayments(),
                paymentTracker.getTotalPot()
            );
        }
        
        System.out.println("+--------------------------------------------------+");
        System.out.printf("| TOTAL POT: |", paymentTracker.getTotalPot());
        System.out.printf("| TOTAL MEMBERS: %39d |\n", memberManager.getMemberCount());
        System.out.printf("| BUDGET GOAL: ", paymentTracker.getBudgetGoal());
        System.out.println("+==================================================+");
    }
    
    /**
     * Case 7: Set budget goal
     */
    private static void setBudgetGoal() {
        System.out.println("\n--- SET BUDGET GOAL ---");
        
        System.out.print("Enter budget goal (P): ");
        double goal = scanner.nextDouble();
        scanner.nextLine();
        
        if (goal <= 0) {
            System.out.println("Goal must be positive!");
            return;
        }
        
        paymentTracker.setBudgetGoal(goal);
        System.out.printf("Budget ", goal);
        
        // Show progress
        double progress = (paymentTracker.getTotalPot() / goal) * 100;
        System.out.printf("Current progress: ", progress);
        
        if (progress >= 100) {
            System.out.println("Congratulations! You've reached your goal!");
        } else {
            double remaining = goal - paymentTracker.getTotalPot();
            System.out.printf("Remaining to reach goal:", remaining);
        }
    }
}

