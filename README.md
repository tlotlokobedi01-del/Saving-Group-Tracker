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
The Savings Group Tracker is designed to assist members in managing a savings group effectively. The application allows users to track contributions, manage budgets, and generate financial reports.

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
├── Main.java
│
├── model/
│   ├── Account.java           ← ABSTRACT CLASS
│   ├── Contributable.java     ← INTERFACE
│   ├── SavingsAccount.java    ← extends Account implements Contributable
│   ├── Transaction.java       ← immutable contribution record
│   └── User.java              ← member details + owns SavingsAccount
│
├── service/
│   ├── MemberManager.java     ← add/find/list members
│   ├── PaymentTracker.java    ← pot, transactions, budget goal
│   └── ReportGenerator.java   ← all print/display logic
│
├── utils/
│   └── FileHandler.java       ← save/load data using Scanner
│
├── data/
│   └── savings.txt            ← auto-generated on first save
│
└── README.md

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

/**
 * Finance Tracker - Community Savings Group
 * CSI142 Project
 * ONLY menu loop is here - all logic is in separate classes
 */
import service.MemberManager;
import service.PaymentTracker;
import service.ReportGenerator;
import model.User;
import utils.FileHandler;
import java.util.Scanner;

public class Main {
    private static MemberManager memberManager = new MemberManager();
    private static PaymentTracker paymentTracker = new PaymentTracker();
    private static ReportGenerator reportGenerator = new ReportGenerator();
    private static Scanner scanner = new Scanner(System.in);

    public static void main(String[] args) {
        FileHandler.load(memberManager, paymentTracker);
        
        while (true) {
            System.out.println("\n=== SAVINGS TRACKER ===");
            System.out.println("1. Add Member");
            System.out.println("2. Record Payment");
            System.out.println("3. Payment Status");
            System.out.println("4. View Total Pot");
            System.out.println("5. Next Receiver");
            System.out.println("6. Full Report");
            System.out.println("7. Set Budget Goal");
            System.out.println("8. Save & Exit");
            System.out.print("Choice: ");
            
            int choice = scanner.nextInt();
            scanner.nextLine();
            
            if (choice == 1) {
                System.out.print("Name: ");
                String name = scanner.nextLine();
                System.out.print("Student ID: ");
                String id = scanner.nextLine();
                memberManager.addMember(name, id);
            }
            else if (choice == 2) {
                System.out.print("Member name: ");
                String name = scanner.nextLine();
                User u = memberManager.findByName(name);
                if (u != null) {
                    System.out.print("Amount: P");
                    double amt = scanner.nextDouble();
                    scanner.nextLine();
                    paymentTracker.recordPayment(u, amt);
                } else {
                    System.out.println("Member not found.");
                }
            }
            else if (choice == 3) {
                if (memberManager.getMemberCount() > 0) {
                    reportGenerator.printPaymentStatus(memberManager.getAllMembers());
                } else {
                    System.out.println("No members.");
                }
            }
            else if (choice == 4) {
                System.out.printf("Total Pot: P%.2f\n", paymentTracker.getTotalPot());
                if (paymentTracker.getBudgetGoal() > 0) {
                    System.out.printf("Goal Progress: %.1f%%\n", paymentTracker.getProgress());
                }
            }
            else if (choice == 5) {
                String next = memberManager.getNextReceiver();
                System.out.println("Next to receive: " + next);
                if (!next.equals("All members have received.")) {
                    System.out.print("Mark as received? (y/n): ");
                    if (scanner.nextLine().equalsIgnoreCase("y")) {
                        memberManager.markAsReceived(next);
                        System.out.println(next + " marked as received!");
                    }
                }
            }
            else if (choice == 6) {
                if (memberManager.getMemberCount() > 0) {
                    reportGenerator.printFullReport(
                        memberManager.getAllMembers(),
                        paymentTracker.getTotalPot(),
                        paymentTracker.getBudgetGoal()
                    );
                    reportGenerator.printTransactionHistory(paymentTracker.getAllTransactions());
                } else {
                    System.out.println("No members.");
                }
            }
            else if (choice == 7) {
                System.out.print("Enter budget goal: P");
                double goal = scanner.nextDouble();
                scanner.nextLine();
                paymentTracker.setBudgetGoal(goal);
                System.out.printf("Goal set to P%.2f\n", goal);
            }
            else if (choice == 8) {
                FileHandler.save(memberManager, paymentTracker);
                System.out.println("Goodbye!");
                break;
            }
        }
        scanner.close();
    }
}

*Submitted for CSI142 Milestone 2 – 15 April 2026*

package model;

public abstract class Account {
    protected String accountId;
    protected String ownerName;
    protected double balance;

    public Account(String accountId, String ownerName) {
        this.accountId = accountId;
        this.ownerName = ownerName;
        this.balance = 0.0;
    }

    public String getAccountId() {
        return accountId;
    }

    public String getOwnerName() {
        return ownerName;
    }

    public double getBalance() {
        return balance;
    }

    public void deposit(double amount) {
        if (amount > 0) {
            balance += amount;
        }
    }

    public abstract String getSummary();
}

package model;

public interface Contributable {
    void contribute(double amount);
    boolean hasReceivedPot();
    void markAsReceived();
    String getContributionStatus();
}

package model;

public class SavingsAccount extends Account implements Contributable {
    private double totalContributed;
    private boolean receivedPot;
    private int turnOrder;
    private double expectedPerCycle;

    public SavingsAccount(String accountId, String ownerName, int turnOrder, double expectedPerCycle) {
        super(accountId, ownerName);
        this.totalContributed = 0.0;
        this.receivedPot = false;
        this.turnOrder = turnOrder;
        this.expectedPerCycle = expectedPerCycle;
    }

    public double getTotalContributed() {
        return totalContributed;
    }

    public int getTurnOrder() {
        return turnOrder;
    }

    public void setExpectedPerCycle(double amount) {
        this.expectedPerCycle = amount;
    }

    public boolean hasPaid() {
        return totalContributed >= expectedPerCycle;
    }

    @Override
    public void contribute(double amount) {
        if (amount > 0) {
            totalContributed += amount;
            deposit(amount);
        }
    }

    @Override
    public boolean hasReceivedPot() {
        return receivedPot;
    }

    @Override
    public void markAsReceived() {
        receivedPot = true;
    }

    @Override
    public String getContributionStatus() {
        String status = hasPaid() ? "PAID" : "OWES";
        String turn = receivedPot ? "Received" : "Waiting (#" + turnOrder + ")";
        return String.format("%-16s | %-6s | P%-8.2f | %s", ownerName, status, totalContributed, turn);
    }

    @Override
    public String getSummary() {
        return String.format("Account %-8s | %-16s | Balance: P%.2f", accountId, ownerName, balance);
    }

    public void resetCycle() {
        totalContributed = 0.0;
        receivedPot = false;
        balance = 0.0;
    }
}

package model;

public class Transaction {
    private final String memberName;
    private final double amount;
    private final String timestamp;

    public Transaction(String memberName, double amount, String timestamp) {
        this.memberName = memberName;
        this.amount = amount;
        this.timestamp = timestamp;
    }

    public String getMemberName() {
        return memberName;
    }

    public double getAmount() {
        return amount;
    }

    public String getTimestamp() {
        return timestamp;
    }

    public String toString() {
        return String.format("[%s] %-16s +P%.2f", timestamp, memberName, amount);
    }
}

package model;

public class User {
    private String name;
    private String studentId;
    private SavingsAccount account;

    public User(String name, String studentId, int turnOrder, double expectedPerCycle) {
        this.name = name;
        this.studentId = studentId;
        this.account = new SavingsAccount(studentId, name, turnOrder, expectedPerCycle);
    }

    public String getName() {
        return name;
    }

    public String getStudentId() {
        return studentId;
    }

    public SavingsAccount getAccount() {
        return account;
    }

    public void contribute(double amount) {
        account.contribute(amount);
    }

    public double getTotalContributed() {
        return account.getTotalContributed();
    }
}

package service;

import model.User;

public class MemberManager {
    private User[] members = new User[50];
    private int count = 0;
    private double expectedPerCycle = 100.0;

    public void addMember(String name, String id) {
        if (count >= 50) {
            System.out.println("Group full.");
            return;
        }
        members[count] = new User(name, id, count + 1, expectedPerCycle);
        count++;
        System.out.println("Member added: " + name);
    }

    public User findByName(String name) {
        for (int i = 0; i < count; i++) {
            if (members[i].getName().equalsIgnoreCase(name)) {
                return members[i];
            }
        }
        return null;
    }

    public User[] getAllMembers() {
        User[] copy = new User[count];
        for (int i = 0; i < count; i++) {
            copy[i] = members[i];
        }
        return copy;
    }

    public int getMemberCount() {
        return count;
    }

    public String getNextReceiver() {
        for (int i = 0; i < count; i++) {
            if (!members[i].getAccount().hasReceivedPot()) {
                return members[i].getName();
            }
        }
        return "All members have received.";
    }

    public void markAsReceived(String name) {
        User u = findByName(name);
        if (u != null) {
            u.getAccount().markAsReceived();
        }
    }

    public double getExpectedPerCycle() {
        return expectedPerCycle;
    }

    public void setExpectedPerCycle(double amount) {
        this.expectedPerCycle = amount;
        for (int i = 0; i < count; i++) {
            members[i].getAccount().setExpectedPerCycle(amount);
        }
    }
}

package service;

import model.Transaction;
import model.User;

public class PaymentTracker {
    private Transaction[] transactions = new Transaction[500];
    private int txCount = 0;
    private double totalPot = 0.0;
    private double budgetGoal = 0.0;
    private int txSeq = 1;

    public void recordPayment(User user, double amount) {
        user.contribute(amount);
        totalPot += amount;
        String ts = "T" + txSeq++;
        transactions[txCount++] = new Transaction(user.getName(), amount, ts);
        System.out.printf("Recorded P%.2f from %s\n", amount, user.getName());
    }

    public double getTotalPot() {
        return totalPot;
    }

    public double getBudgetGoal() {
        return budgetGoal;
    }

    public void setBudgetGoal(double goal) {
        this.budgetGoal = goal;
    }

    public void setTotalPot(double pot) {
        this.totalPot = pot;
    }

    public Transaction[] getAllTransactions() {
        Transaction[] copy = new Transaction[txCount];
        for (int i = 0; i < txCount; i++) {
            copy[i] = transactions[i];
        }
        return copy;
    }

    public double getProgress() {
        if (budgetGoal <= 0) return 0;
        return (totalPot / budgetGoal) * 100.0;
    }
}

package service;

import model.User;
import model.Transaction;

public class ReportGenerator {

    public void printPaymentStatus(User[] members) {
        System.out.println("\n--- PAYMENT STATUS ---");
        System.out.println("----------------------------------------");
        for (User u : members) {
            System.out.println(u.getAccount().getContributionStatus());
        }
    }

    public void printFullReport(User[] members, double totalPot, double budgetGoal) {
        System.out.println("\n========== FULL REPORT ==========");
        for (int i = 0; i < members.length; i++) {
            System.out.println((i+1) + ". " + members[i].getAccount().getSummary());
        }
        System.out.println("-----------------------------------");
        System.out.printf("Total Pot: P%.2f\n", totalPot);
        System.out.printf("Members: %d\n", members.length);
        if (budgetGoal > 0) {
            System.out.printf("Goal: P%.2f (%.1f%% reached)\n", budgetGoal, (totalPot/budgetGoal)*100);
        }
        System.out.println("===================================");
    }

    public void printTransactionHistory(Transaction[] txs) {
        System.out.println("\n--- TRANSACTIONS ---");
        if (txs.length == 0) {
            System.out.println("No transactions yet.");
            return;
        }
        for (Transaction t : txs) {
            System.out.println(t);
        }
    }
}

package utils;

import service.MemberManager;
import service.PaymentTracker;
import model.User;
import java.io.Expection;
import java.util.Scanner;

public class FileHandler {
    private static final String FILE = "data/savings.txt";

    public static void save(MemberManager manager, PaymentTracker tracker) {
        try {
            new File("data").mkdirs();
            PrintWriter pw = new PrintWriter(FILE);
            
            pw.println("EXPECTED=" + manager.getExpectedPerCycle());
            pw.println("TOTAL_POT=" + tracker.getTotalPot());
            pw.println("BUDGET_GOAL=" + tracker.getBudgetGoal());
            
            for (User u : manager.getAllMembers()) {
                pw.println(u.getName() + "|" + u.getStudentId() + "|" + 
                          u.getTotalContributed() + "|" + u.getAccount().hasReceivedPot());
            }
            pw.close();
            System.out.println("Saved.");
        } catch (IOException e) {
            System.out.println("Save failed.");
        }
    }

    public static void load(MemberManager manager, PaymentTracker tracker) {
        File f = new File(FILE);
        if (!f.exists()) return;

        try (Scanner sc = new Scanner(f)) {
            while (sc.hasNextLine()) {
                String line = sc.nextLine().trim();
                if (line.isEmpty()) continue;
                
                if (line.startsWith("EXPECTED=")) {
                    double val = Double.parseDouble(line.split("=")[1]);
                    manager.setExpectedPerCycle(val);
                }
                else if (line.startsWith("TOTAL_POT=")) {
                    double val = Double.parseDouble(line.split("=")[1]);
                    tracker.setTotalPot(val);
                }
                else if (line.startsWith("BUDGET_GOAL=")) {
                    double val = Double.parseDouble(line.split("=")[1]);
                    tracker.setBudgetGoal(val);
                }
                else if (line.contains("|")) {
                    String[] p = line.split("\\|");
                    manager.addMember(p[0], p[1]);
                    User u = manager.findByName(p[0]);
                    if (u != null) {
                        u.contribute(Double.parseDouble(p[2]));
                        if (Boolean.parseBoolean(p[3])) {
                            u.getAccount().markAsReceived();
                        }
                    }
                }
            }
            System.out.println("Loaded.");
        } catch (FileNotFoundException e) {
            System.out.println("No save file found.");
        }
    }
}

*Submitted for CSI142 Milestone 3 – 22 April 2026*
