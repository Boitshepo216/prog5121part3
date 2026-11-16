# prog5121part3

/*
 * Click nbfs://nbhost/SystemFileSystem/Templates/Licenses/license-default.txt to change this license
 */

package com.mycompany.prog5121part3;

/**
 *
 * @author Feniu
 */
import javax.swing.JOptionPane;
import java.io.BufferedReader;
import java.io.File;
import java.io.FileReader;
import java.io.FileWriter;
import java.io.IOException;
import java.util.ArrayList;
import java.util.List;
import java.util.Random;

public class Prog5121part3 {

    // Stores registered user accounts using your Login class
    private static final List<Login> registeredUsers = new ArrayList<>();
    private static String currentLoggedInUser = "";

    // Part 3: Dynamically populated arrays/lists (no hard-coding)
    private static final ArrayList<String> sentMessages = new ArrayList<>();
    private static final ArrayList<String> disregardedMessages = new ArrayList<>();
    private static final ArrayList<String> storedMessages = new ArrayList<>();
    private static final ArrayList<String> messageHashes = new ArrayList<>();
    private static final ArrayList<String> messageIDs = new ArrayList<>();

    // Parallel tracking lists to associate sender/recipient/status with each message
    private static final ArrayList<String> allSenders = new ArrayList<>();
    private static final ArrayList<String> allRecipients = new ArrayList<>();
    private static final ArrayList<String> allStatuses = new ArrayList<>();

    private static int messageSequence = 0;
    private static final String JSON_FILE_PATH = "stored_messages.json";

    public static void main(String[] args) {
        JOptionPane.showMessageDialog(null, "Welcome to QuickChat.");

        boolean appRunning = true;
        while (appRunning) {
            String[] authOptions = {"Register", "Login", "Exit"};
            int authChoice = JOptionPane.showOptionDialog(
                    null,
                    "Select an option to continue:",
                    "QuickChat Authentication",
                    JOptionPane.DEFAULT_OPTION,
                    JOptionPane.PLAIN_MESSAGE,
                    null,
                    authOptions,
                    authOptions[0]
            );

            switch (authChoice) {
                case 0 -> processRegistration(registeredUsers);
                case 1 -> {
                    if (processLogin(registeredUsers)) {
                        loadStoredMessagesFromJSON();
                        runChatMainMenu();
                    }
                }
                case 2, JOptionPane.CLOSED_OPTION -> {
                    JOptionPane.showMessageDialog(null, "Exiting application. Goodbye!");
                    appRunning = false;
                }
            }
        }
    }

    // =========================================================================
    // PART 1: REGISTRATION & LOGIN WORKFLOW (Using your Login class)
    // =========================================================================

    private static void processRegistration(List<Login> userDatabase) {
        Login newAccount = new Login();

        // Step 1: Username
        String username = JOptionPane.showInputDialog("Enter username (must contain '_' and <= 5 chars):");
        if (username == null) return;
        username = username.trim();

        if (!newAccount.checkUserName(username)) {
            JOptionPane.showMessageDialog(null, "Username is not correctly formatted, please ensure that your username contains an underscore and is no more than 5 characters in length.");
            return;
        }
        JOptionPane.showMessageDialog(null, "Username successfully captured.");
        newAccount.setUsername(username);

        // Step 2: Password
        String password = JOptionPane.showInputDialog("Enter password (min 8 chars, 1 capital, 1 number, 1 special char):");
        if (password == null) return;

        if (!newAccount.checkPasswordComplexity(password)) {
            JOptionPane.showMessageDialog(null, "Password is not correctly formatted, please ensure that the password contains at least 8 characters, a capital letter, a number and a special character.");
            return;
        }
        JOptionPane.showMessageDialog(null, "Password successfully captured.");
        newAccount.setPassword(password);

        // Step 3: SA Phone Number
        String phone = JOptionPane.showInputDialog("Enter SA phone number (+27xxxxxxxxx):");
        if (phone == null) return;
        phone = phone.trim();

        if (!newAccount.checkPhoneNumber(phone)) {
            JOptionPane.showMessageDialog(null, "Cell phone number incorrectly formatted or does not contain international code.");
            return;
        }
        JOptionPane.showMessageDialog(null, "Cellphone number is successfully added!");
        newAccount.setPhoneNumber(phone);

        // Step 4: Save to user list
        String registrationOutcome = newAccount.registerUser(userDatabase);
        JOptionPane.showMessageDialog(null, registrationOutcome);
    }

    private static boolean processLogin(List<Login> userDatabase) {
        if (userDatabase.isEmpty()) {
            JOptionPane.showMessageDialog(null, "No accounts registered yet. Please register first.");
            return false;
        }

        String loginUser = JOptionPane.showInputDialog("Login - Enter username:");
        if (loginUser == null) return false;
        loginUser = loginUser.trim();

        String loginPass = JOptionPane.showInputDialog("Login - Enter password:");
        if (loginPass == null) return false;

        Login authHandler = new Login();
        boolean loginSuccess = authHandler.loginUser(userDatabase, loginUser, loginPass);
        String statusMessage = authHandler.returnLoginStatus(loginSuccess, loginUser);

        JOptionPane.showMessageDialog(null, statusMessage);

        if (loginSuccess) {
            currentLoggedInUser = loginUser;
            return true;
        }
        return false;
    }

    // =========================================================================
    // PART 2 & 3: MAIN CHAT MENU (Includes 4th Option: Stored Messages)
    // =========================================================================

    private static void runChatMainMenu() {
        boolean loggedIn = true;

        while (loggedIn) {
            String[] menuOptions = {"Send Messages", "Show Recently Sent Messages", "Quit", "Stored Messages"};
            int menuChoice = JOptionPane.showOptionDialog(
                    null,
                    "Welcome to QuickChat.\nChoose an option from the menu:",
                    "QuickChat Main Menu",
                    JOptionPane.DEFAULT_OPTION,
                    JOptionPane.INFORMATION_MESSAGE,
                    null,
                    menuOptions,
                    menuOptions[0]
            );

            switch (menuChoice) {
                case 0 -> handleSendingMessages();
                case 1 -> JOptionPane.showMessageDialog(null, "Coming Soon.");
                case 2, JOptionPane.CLOSED_OPTION -> {
                    loggedIn = false;
                    JOptionPane.showMessageDialog(null, "Logged out of QuickChat.");
                }
                case 3 -> handleStoredMessagesMenu();
            }
        }
    }

    private static void handleSendingMessages() {
        String countStr = JOptionPane.showInputDialog("How many messages would you like to enter?");
        if (countStr == null) return;

        int messageLimit;
        try {
            messageLimit = Integer.parseInt(countStr.trim());
            if (messageLimit <= 0) {
                JOptionPane.showMessageDialog(null, "Please enter a number greater than 0.");
                return;
            }
        } catch (NumberFormatException e) {
            JOptionPane.showMessageDialog(null, "Invalid number entered.");
            return;
        }

        for (int i = 0; i < messageLimit; i++) {
            String recipient = JOptionPane.showInputDialog("Enter recipient cell number (+27xxxxxxxxx):");
            if (recipient == null) return;
            recipient = recipient.trim();

            Message msgObj = new Message();
            if (!msgObj.checkRecipientCell(recipient)) {
                JOptionPane.showMessageDialog(null, "Cell phone number incorrectly formatted or does not contain international code.");
                i--;
                continue;
            }

            String messageText = JOptionPane.showInputDialog("Enter your message (max 250 characters):");
            if (messageText == null) return;

            if (messageText.length() > 250) {
                JOptionPane.showMessageDialog(null, "Please enter a message of less than 250 characters.");
                i--;
                continue;
            }

            messageSequence++;
            msgObj.setRecipient(recipient);
            msgObj.setMessageText(messageText);
            msgObj.setMessageNumber(messageSequence);
            msgObj.generateMessageID();
            msgObj.createMessageHash();

            String actionResult = msgObj.SentMessage();

            switch (actionResult) {
                case "SEND" -> {
                    sentMessages.add(msgObj.getMessageText());
                    messageIDs.add(msgObj.getMessageID());
                    messageHashes.add(msgObj.getMessageHash());
                    allSenders.add(currentLoggedInUser);
                    allRecipients.add(msgObj.getRecipient());
                    allStatuses.add("Sent");

                    JOptionPane.showMessageDialog(null, msgObj.printMessages());
                    JOptionPane.showMessageDialog(null, "Message successfully sent.");
                }
                case "DISREGARD" -> {
                    disregardedMessages.add(msgObj.getMessageText());
                    JOptionPane.showMessageDialog(null, "Message disregarded.");
                }
                case "STORE" -> {
                    storedMessages.add(msgObj.getMessageText());
                    messageIDs.add(msgObj.getMessageID());
                    messageHashes.add(msgObj.getMessageHash());
                    allSenders.add(currentLoggedInUser);
                    allRecipients.add(msgObj.getRecipient());
                    allStatuses.add("Stored");

                    msgObj.storeMessage(currentLoggedInUser, JSON_FILE_PATH);
                    JOptionPane.showMessageDialog(null, "Message successfully stored.");
                }
            }
        }

        JOptionPane.showMessageDialog(null, "Total messages sent: " + Message.returnTotalMessagess(sentMessages));
    }

    // =========================================================================
    // PART 3: JSON FILE READER & STORED MESSAGES SUB-MENU (2a - 2f)
    // =========================================================================

    private static void loadStoredMessagesFromJSON() {
        File jsonFile = new File(JSON_FILE_PATH);
        if (!jsonFile.exists()) return;

        try (BufferedReader reader = new BufferedReader(new FileReader(jsonFile))) {
            String line;
            while ((line = reader.readLine()) != null) {
                if (line.trim().isEmpty()) continue;

                String id = parseJsonValue(line, "MessageID");
                String hash = parseJsonValue(line, "MessageHash");
                String sender = parseJsonValue(line, "Sender");
                String recipient = parseJsonValue(line, "Recipient");
                String text = parseJsonValue(line, "Message");

                if (!id.isEmpty() && !messageIDs.contains(id)) {
                    storedMessages.add(text);
                    messageIDs.add(id);
                    messageHashes.add(hash);
                    allSenders.add(sender);
                    allRecipients.add(recipient);
                    allStatuses.add("Stored");
                }
            }
        } catch (IOException e) {
            JOptionPane.showMessageDialog(null, "Error reading stored messages file: " + e.getMessage());
        }
    }

    private static String parseJsonValue(String jsonLine, String key) {
        String token = "\"" + key + "\":\"";
        int startIndex = jsonLine.indexOf(token);
        if (startIndex == -1) return "";
        startIndex += token.length();
        int endIndex = jsonLine.indexOf("\"", startIndex);
        return (endIndex == -1) ? "" : jsonLine.substring(startIndex, endIndex);
    }

    private static void handleStoredMessagesMenu() {
        loadStoredMessagesFromJSON();

        String subMenuPrompt = """
                Stored Messages & Reports Menu:
                a. Display sender and recipient of all stored/sent messages
                b. Display the longest message
                c. Search for a message ID
                d. Search all messages for a particular recipient
                e. Delete a message using the message hash
                f. Display a full report of all messages
                """;

        String selection = JOptionPane.showInputDialog(subMenuPrompt);
        if (selection == null) return;

        switch (selection.trim().toLowerCase()) {
            case "a" -> displaySendersAndRecipients();
            case "b" -> displayLongestMessage();
            case "c" -> searchByMessageID();
            case "d" -> searchByRecipient();
            case "e" -> deleteByMessageHash();
            case "f" -> displayFullReport();
            default -> JOptionPane.showMessageDialog(null, "Invalid option. Please choose a, b, c, d, e, or f.");
        }
    }

    // 2a. Display the sender and recipient of all stored/sent messages
    private static void displaySendersAndRecipients() {
        if (allRecipients.isEmpty()) {
            JOptionPane.showMessageDialog(null, "No messages found.");
            return;
        }

        StringBuilder sb = new StringBuilder("--- Senders and Recipients ---\n");
        for (int i = 0; i < allRecipients.size(); i++) {
            sb.append("Sender: ").append(allSenders.get(i))
              .append(" | Recipient: ").append(allRecipients.get(i))
              .append(" [").append(allStatuses.get(i)).append("]\n");
        }
        JOptionPane.showMessageDialog(null, sb.toString());
    }

    // 2b. Display the longest stored/sent message
    private static void displayLongestMessage() {
        ArrayList<String> combined = new ArrayList<>();
        combined.addAll(sentMessages);
        combined.addAll(storedMessages);

        if (combined.isEmpty()) {
            JOptionPane.showMessageDialog(null, "No messages available.");
            return;
        }

        String longestMsg = "";
        for (String msg : combined) {
            if (msg.length() > longestMsg.length()) {
                longestMsg = msg;
            }
        }
        JOptionPane.showMessageDialog(null, "Longest Message:\n\"" + longestMsg + "\"");
    }

    // 2c. Search for a message ID and display the corresponding recipient and message
    private static void searchByMessageID() {
        String searchId = JOptionPane.showInputDialog("Enter the 10-digit Message ID:");
        if (searchId == null || searchId.trim().isEmpty()) return;

        for (int i = 0; i < messageIDs.size(); i++) {
            if (messageIDs.get(i).equals(searchId.trim())) {
                String content = resolveMessageText(i);
                JOptionPane.showMessageDialog(null,
                        "Message Found:\nRecipient: " + allRecipients.get(i) +
                        "\nMessage: \"" + content + "\"");
                return;
            }
        }
        JOptionPane.showMessageDialog(null, "No message found with ID: " + searchId);
    }

    // 2d. Search for all the messages sent or stored for a particular recipient
    private static void searchByRecipient() {
        String targetRecipient = JOptionPane.showInputDialog("Enter recipient cell number (+27xxxxxxxxx):");
        if (targetRecipient == null || targetRecipient.trim().isEmpty()) return;

        StringBuilder matches = new StringBuilder("Messages for " + targetRecipient + ":\n");
        boolean foundAny = false;

        for (int i = 0; i < allRecipients.size(); i++) {
            if (allRecipients.get(i).equals(targetRecipient.trim())) {
                matches.append("- \"").append(resolveMessageText(i)).append("\"\n");
                foundAny = true;
            }
        }

        if (foundAny) {
            JOptionPane.showMessageDialog(null, matches.toString());
        } else {
            JOptionPane.showMessageDialog(null, "No messages found for recipient: " + targetRecipient);
        }
    }

    // 2e. Delete a message using the message hash
    private static void deleteByMessageHash() {
        String targetHash = JOptionPane.showInputDialog("Enter the Message Hash to delete (e.g., 00:1:HITHANKS):");
        if (targetHash == null || targetHash.trim().isEmpty()) return;

        for (int i = 0; i < messageHashes.size(); i++) {
            if (messageHashes.get(i).equalsIgnoreCase(targetHash.trim())) {
                String deletedMessage = resolveMessageText(i);

                sentMessages.remove(deletedMessage);
                storedMessages.remove(deletedMessage);

                messageIDs.remove(i);
                messageHashes.remove(i);
                allSenders.remove(i);
                allRecipients.remove(i);
                allStatuses.remove(i);

                JOptionPane.showMessageDialog(null, "Message: \"" + deletedMessage + "\" successfully deleted.");
                return;
            }
        }
        JOptionPane.showMessageDialog(null, "Message hash not found.");
    }

    // 2f. Display a report that lists the full details of all stored/sent messages
    private static void displayFullReport() {
        if (messageIDs.isEmpty()) {
            JOptionPane.showMessageDialog(null, "No messages available to display.");
            return;
        }

        StringBuilder report = new StringBuilder("=== FULL MESSAGE REPORT ===\n\n");
        for (int i = 0; i < messageIDs.size(); i++) {
            report.append("Message ID: ").append(messageIDs.get(i)).append("\n")
                  .append("Message Hash: ").append(messageHashes.get(i)).append("\n")
                  .append("Sender: ").append(allSenders.get(i)).append("\n")
                  .append("Recipient: ").append(allRecipients.get(i)).append("\n")
                  .append("Status: ").append(allStatuses.get(i)).append("\n")
                  .append("Message: ").append(resolveMessageText(i)).append("\n")
                  .append("----------------------------------\n");
        }
        JOptionPane.showMessageDialog(null, report.toString());
    }

    private static String resolveMessageText(int parallelIndex) {
        String status = allStatuses.get(parallelIndex);
        int count = 0;
        for (int j = 0; j < parallelIndex; j++) {
            if (allStatuses.get(j).equals(status)) {
                count++;
            }
        }
        if ("Sent".equals(status) && count < sentMessages.size()) {
            return sentMessages.get(count);
        } else if ("Stored".equals(status) && count < storedMessages.size()) {
            return storedMessages.get(count);
        }
        return "";
    }

    // =========================================================================
    // NESTED LOGIN CLASS (Your exact Part 1 class structure)
    // =========================================================================
    public static class Login {

        private String username;
        private String password;
        private String phoneNumber;

        public Login() {}

        public Login(String username, String password, String phoneNumber) {
            this.username = username;
            this.password = password;
            this.phoneNumber = phoneNumber;
        }

        public boolean checkUserName(String username) {
            return username != null && username.contains("_") && username.length() <= 5;
        }

        public boolean checkPasswordComplexity(String password) {
            if (password == null || password.length() < 8) {
                return false;
            }
            boolean hasUppercase = !password.equals(password.toLowerCase());
            boolean hasNumber = password.matches(".*\\d.*");
            boolean hasSpecialChar = password.matches(".*[!@#$%^&*(),.?\"{}|<>].*");

            return hasUppercase && hasNumber && hasSpecialChar;
        }

        public boolean checkPhoneNumber(String phoneNumber) {
            if (phoneNumber == null) return false;
            String saPhoneRegex = "^\\+27[6-8]\\d{8}$";
            return phoneNumber.matches(saPhoneRegex);
        }

        public String registerUser(List<Login> userList) {
            for (Login existingUser : userList) {
                if (existingUser.getUsername() != null && existingUser.getUsername().equalsIgnoreCase(this.username)) {
                    return "Username already exists. Please choose another username.";
                }
            }
            userList.add(this);
            return "User registered successfully!";
        }

        public boolean loginUser(List<Login> userList, String inputUsername, String inputPassword) {
            for (Login account : userList) {
                if (account.getUsername().equals(inputUsername) && account.getPassword().equals(inputPassword)) {
                    return true;
                }
            }
            return false;
        }

        public String returnLoginStatus(boolean isLoggedIn, String username) {
            if (isLoggedIn) {
                return "A successful login! Welcome back " + username + ", it is great to see you again.";
            } else {
                return "A failed login! Username or password incorrect, please try again.";
            }
        }

        public String getUsername() { return username; }
        public void setUsername(String username) { this.username = username; }
        public String getPassword() { return password; }
        public void setPassword(String password) { this.password = password; }
        public String getPhoneNumber() { return phoneNumber; }
        public void setPhoneNumber(String phoneNumber) { this.phoneNumber = phoneNumber; }
    }

    // =========================================================================
    // NESTED MESSAGE CLASS (Handles Part 2 & Part 3 message methods)
    // =========================================================================
    public static class Message {

        private String messageID;
        private int messageNumber;
        private String recipient;
        private String messageText;
        private String messageHash;

        public Message() {}

        public void generateMessageID() {
            this.messageID = String.format("%010d", new Random().nextInt(1_000_000_000));
        }

        public boolean checkMessageID() {
            return messageID != null && messageID.length() <= 10;
        }

        public boolean checkRecipientCell(String cell) {
            return cell != null && cell.matches("^\\+27[6-8]\\d{8}$");
        }

        public String createMessageHash() {
            String[] words = messageText.trim().split("\\s+");
            String firstWord = words[0];
            String lastWord = words[words.length - 1];
            String idPrefix = (messageID != null && messageID.length() >= 2) ? messageID.substring(0, 2) : "00";

            this.messageHash = (idPrefix + ":" + messageNumber + ":" + firstWord + lastWord).toUpperCase();
            return this.messageHash;
        }

        public String SentMessage() {
            String[] choices = {"Send Message", "Disregard Message", "Store Message to send later"};
            int choice = JOptionPane.showOptionDialog(
                    null,
                    "Choose an action for this message:",
                    "Message Options",
                    JOptionPane.DEFAULT_OPTION,
                    JOptionPane.QUESTION_MESSAGE,
                    null,
                    choices,
                    choices[0]
            );

            return switch (choice) {
                case 0 -> "SEND";
                case 2 -> "STORE";
                default -> "DISREGARD";
            };
        }

        public String printMessages() {
            return "Message ID: " + messageID +
                   "\nMessage Hash: " + messageHash +
                   "\nRecipient: " + recipient +
                   "\nMessage: " + messageText;
        }

        public static int returnTotalMessagess(ArrayList<String> sentList) {
            return sentList.size();
        }

        public void storeMessage(String sender, String filePath) {
            String escapedMsg = messageText.replace("\"", "\\\"");
            String jsonRecord = "{\"MessageID\":\"" + messageID + "\"," +
                                "\"MessageHash\":\"" + messageHash + "\"," +
                                "\"Sender\":\"" + sender + "\"," +
                                "\"Recipient\":\"" + recipient + "\"," +
                                "\"Message\":\"" + escapedMsg + "\"}\n";

            try (FileWriter writer = new FileWriter(filePath, true)) {
                writer.write(jsonRecord);
            } catch (IOException e) {
                JOptionPane.showMessageDialog(null, "Error storing message to JSON: " + e.getMessage());
            }
        }

        // Getters and Setters
        public String getMessageID() { return messageID; }
        public void setMessageNumber(int messageNumber) { this.messageNumber = messageNumber; }
        public String getRecipient() { return recipient; }
        public void setRecipient(String recipient) { this.recipient = recipient; }
        public String getMessageText() { return messageText; }
        public void setMessageText(String messageText) { this.messageText = messageText; }
        public String getMessageHash() { return messageHash; }
    }
}
