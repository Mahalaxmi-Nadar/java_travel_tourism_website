# java_travel_tourism_website
created a website using java based on travel and tourism in addition with database insight

import java.sql.*;
import java.util.Date;
import java.util.Scanner;

public class TravelManagementSystem {
    private static final String URL = "jdbc:mysql://localhost:3306/travelmanagement";
    private static final String USERNAME = "root";
    private static final String PASSWORD = "Harsh@409";
    private static final Scanner scanner = new Scanner(System.in);
    private static int currentUserId; // Variable to store the current user's ID

    public static void main(String[] args) {
        try (Connection connection = DriverManager.getConnection(URL, USERNAME, PASSWORD)) {
            System.out.print("Welcome to Travel Management System!");
            //Delay of 0.5s
            for(int i=0;i<5;i++) {
                try {
                    System.out.print(".");
                    Thread.sleep(500);
                } catch (InterruptedException e) {
                    System.out.println( e.getMessage());
                }
            }
            System.out.println();


            boolean loggedIn = false;

            System.out.println("Enter 'L' to login or 'R' to register:");
            String input = scanner.next().toUpperCase();

            while (!loggedIn) {
                if (input.equals("L")) {
                    loggedIn = handleLogin(connection);
                } else if (input.equals("R")) {
                    registerCustomer(connection);
                    loggedIn = true; // Automatically logs in after registration
                } else {
                    System.out.println("Invalid input. Enter 'L' to login or 'R' to register:");
                    input = scanner.next().toUpperCase();
                }
            }

            // Determine whether the logged-in user is an admin or a regular user
            boolean isAdmin = checkAdminStatus(connection);

            // Perform actions based on user role
            while (true) {
                if (isAdmin) adminMenu(connection);
                else {
                    userMenu(connection);
                }
            }

        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    // Method to handle user login
    private static boolean handleLogin(Connection connection) {
        try {
            System.out.println("Enter username:");
            String username = scanner.next();
            System.out.println("Enter password:");
            String password = scanner.next();

            PreparedStatement preparedStatement = connection.prepareStatement(
                    "SELECT * FROM user WHERE username = ? AND password = ?");
            preparedStatement.setString(1, username);
            preparedStatement.setString(2, password);
            ResultSet resultSet = preparedStatement.executeQuery();

            if (resultSet.next()) {
                currentUserId = resultSet.getInt("id");
                System.out.println("Login successful");
                return true;
            } else {
                System.out.println("Invalid username or password");
                return false;
            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
            return false;
        }
    }

    // Method to register a new customer
    private static void registerCustomer(Connection connection) {
        try {
            System.out.println("Enter new username:");
            String username = scanner.next();
            System.out.println("Enter new password:");
            String password = scanner.next();

            PreparedStatement  preparedStatement = connection.prepareStatement(
                    "INSERT INTO user (username, password, isAdmin) VALUES (username,password, false)");
            preparedStatement.setString(1, username);
            preparedStatement.setString(2, password);
            int rowsAffected = preparedStatement.executeUpdate();

            if (rowsAffected > 0) {
                System.out.println("Registration successful. You can now login.");
            } else {
                System.out.println("Registration failed. Please try again.");
            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    // Method to check if the logged-in user is an admin
    private static boolean checkAdminStatus(Connection connection) {
        try {
            PreparedStatement preparedStatement = connection.prepareStatement(
                    "SELECT isAdmin FROM user WHERE id = ?");
            preparedStatement.setInt(1, currentUserId);
            ResultSet resultSet = preparedStatement.executeQuery();

            if (resultSet.next()) {
                return resultSet.getBoolean("isAdmin");
            } else {
                System.out.println("User not found");
                return false;
            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
            return false;
        }
    }

    // Method to display the admin menu
    private static void adminMenu(Connection connection) {
        while (true) {
            System.out.println("Admin menu options:");
            System.out.println("1. Add Destination");
            System.out.println("2. Remove Destination");
            System.out.println("3. Add Package");
            System.out.println("4. Remove Package");
            System.out.println("5. View All Bookings");
            System.out.println("6. Logout");
            int choice;
            choice = scanner.nextInt();
            switch (choice) {
                case 1:
                    addDestination(connection);
                    break;
                case 2:
                    removeDestination(connection);
                    break;
                case 3:
                    addPackage(connection);
                    break;
                case 4:
                    removePackage(connection);
                    break;
                case 5:
                    viewAllBookings(connection);
                    break;
                case 6:
                    System.exit(0);
                default:
                    System.out.println("Invalid choice");
            }
        }
    }

    // Method to add a destination
    private static void addDestination(Connection connection) {
        try {
            System.out.println("Enter destination name:");
            String name = scanner.next();
            System.out.println("Enter destination description:");
            String description = scanner.next();


            String query = "INSERT INTO destination (name, description) VALUES (?, ?)";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setString(1, name);
            preparedStatement.setString(2, description);
            preparedStatement.executeUpdate();
            System.out.println("Destination added successfully.");
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }

    // Method to remove a destination
    private static void removeDestination(Connection connection) {
        try {
            System.out.println("Enter destination ID to remove:");
            int destinationId = scanner.nextInt();

            String query = "DELETE FROM destination WHERE id = ?";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setInt(1, destinationId);
            int rowsAffected = preparedStatement.executeUpdate();
            if (rowsAffected > 0) {
                System.out.println("Destination removed successfully.");
            } else {
                System.out.println("Destination not found.");
            }
        } catch (SQLException e) {
            System.out.println(e.getMessage());
        }
    }

    // Method to add a package
    private static void addPackage(Connection connection) {
        try {
            System.out.println("Enter package name:");
            String name = scanner.next();
            System.out.println("Enter package description:");
            String description = scanner.next();
            System.out.println("Enter package price:");
            double price = scanner.nextDouble();

            String query = "INSERT INTO Package (name, description, price) VALUES (?, ?, ?)";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setString(1, name);
            preparedStatement.setString(2, description);
            preparedStatement.setDouble(3, price);
            preparedStatement.executeUpdate();
            System.out.println("Package added successfully.");
        } catch (SQLException e) {
            System.out.println( e.getMessage());
        }
    }

    // Method to remove a package
    private static void removePackage(Connection connection) {
        try {
            System.out.println("Enter package ID to remove:");
            int packageId = scanner.nextInt();

            String query = "DELETE FROM Package WHERE id = ?";
            PreparedStatement preparedStatement = connection.prepareStatement(query);
            preparedStatement.setInt(1, packageId);
            int rowsAffected = preparedStatement.executeUpdate();
            if (rowsAffected > 0) {
                System.out.println("Package removed successfully.");
            } else {
                System.out.println("Package not found.");
            }
        } catch (SQLException e) {
            System.out.println( e.getMessage());
        }
    }

    // Method to view all bookings
    private static void viewAllBookings(Connection connection) {
        try {
            PreparedStatement statement =  connection.prepareStatement("SELECT * FROM booking");
            ResultSet resultSet = statement.executeQuery();
            System.out.println("All Bookings:");
            System.out.println("+----+--------+-----------+--------------+");
            System.out.println("| ID | UserID | PackageID | DATE         |");
            System.out.println("+----+--------+-----------+--------------+");
            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                int user_id = resultSet.getInt("user_id");
                int package_id = resultSet.getInt("package_id");
                Date booking_date = resultSet.getDate("booking_date");
                System.out.printf("| %-2d | %-6d | %-9s | %-12s |\n", id, user_id, package_id, booking_date);
                System.out.println("+----+--------+-----------+--------------+");
            }
        } catch (SQLException e) {
            System.out.println( e.getMessage());
        }
    }

    // Method to display the user menu
    private static void userMenu(Connection connection) {
        while (true) {
            try {
                System.out.println("User menu options:");
                Thread.sleep(500);
                System.out.println("1. Search Destinations");
                Thread.sleep(500);
                System.out.println("2. viewPackage");
                Thread.sleep(500);
                System.out.println("3. Book Package");
                Thread.sleep(500);
                System.out.println("4. Manage Bookings");
                Thread.sleep(500);
                System.out.println("5. Logout");
            }catch (InterruptedException e){
                System.out.println( e.getMessage());
            }
            int choice = scanner.nextInt();
            switch (choice) {
                case 1:
                    searchDestinations(connection);
                    break;
                case 2:
                    viewPackage(connection);
                    break;
                case 3:
                    bookPackage(connection);
                    break;
                case 4:
                    manageBookings(connection);
                    break;
                case 5:
                    System.exit(0);
                default:
                    System.out.println("Invalid choice");
            }
        }
    }

    // Method to search destinations
    private static void searchDestinations(Connection connection) {
        try {
            PreparedStatement statement = connection.prepareStatement("SELECT * FROM destination");
            ResultSet resultSet = statement.executeQuery();
            System.out.println("Destinations");
            System.out.println("+----+----------------------+------------------------------------------------------+");
            System.out.println("| ID | NAME                 | DESCRIPTION                                          |");
            System.out.println("+----+----------------------+------------------------------------------------------+");


            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String name = resultSet.getString("name");
                String description = resultSet.getString("description");

                System.out.printf("| %-2d | %-20s | %-52s |\n",id, name,description);
                System.out.println("+----+----------------------+------------------------------------------------------+");

            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    private static void viewPackage(Connection connection){
        try {
            PreparedStatement statement = connection.prepareStatement("SELECT * FROM package");
            ResultSet resultSet = statement.executeQuery();
            System.out.println("Package");
            System.out.println("+----+--------------------------+------------------------------------------------------+----------");
            System.out.println("| ID | NAME                     | DESCRIPTION                                          | PRICE    ");
            System.out.println("+----+--------------------------+------------------------------------------------------+----------");


            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                String name = resultSet.getString("name");
                String description = resultSet.getString("description");
                String price = String.valueOf(resultSet.getDouble("price"));
                System.out.printf("| %-2d | %-24s | %-52s | %-8s |\n",id,name,description,price);
                System.out.println("+----+--------------------------+------------------------------------------------------+----------");

            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    // Method to book a package
    private static void bookPackage(Connection connection) {
        System.out.println("Enter Package ID to book:");
        int destinationId = scanner.nextInt();
        try {
            // Use PreparedStatement to insert the current date and time along with user and destination IDs
            PreparedStatement preparedStatement = connection.prepareStatement("INSERT INTO booking (user_id, package_id, booking_date) VALUES (?, ?, NOW())");
            preparedStatement.setInt(1, currentUserId); // Using current user's ID
            preparedStatement.setInt(2, destinationId);
            int rowsAffected = preparedStatement.executeUpdate();
            if (rowsAffected > 0) {
                System.out.println("Booking successful!");
            } else {
                System.out.println("Booking failed.");
            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    // Method to manage bookings
    private static void manageBookings(Connection connection) {
        while (true) {
            System.out.println("Manage Bookings menu:");
            System.out.println("1. View Bookings");
            System.out.println("2. Cancel Booking");
            System.out.println("3. Back to Main Menu");
            int choice = scanner.nextInt();
            switch (choice) {
                case 1:
                    viewBookings(connection);
                    break;
                case 2:
                    cancelBooking(connection);
                    break;
                case 3:
                    return;
                default:
                    System.out.println("Invalid choice");
            }
        }
    }

    // Method to view bookings
    private static void viewBookings(Connection connection) {
        try {
            System.out.println("Your bookings:");
            PreparedStatement preparedStatement = connection.prepareStatement("SELECT * FROM booking WHERE user_id = ?");
            preparedStatement.setInt(1, currentUserId); // Only fetching bookings for the current user
            ResultSet resultSet = preparedStatement.executeQuery();
            System.out.println("viewBooking");
            System.out.println("+----+--------+-----------+--------------+");
            System.out.println("| ID | UserID | PackageID | DATE         |");
            System.out.println("+----+--------+-----------+--------------+");


            while (resultSet.next()) {
                int id = resultSet.getInt("id");
                int user_id = resultSet.getInt("user_id");
                int package_id = resultSet.getInt("package_id");
                Date booking_date = resultSet.getDate("booking_date");
                System.out.printf("| %-2d | %-6d | %-9s | %-12s |\n", id,user_id, package_id,booking_date);
                System.out.println("+----+--------+-----------+--------------+");

            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

    // Method to cancel a booking
    private static void cancelBooking(Connection connection) {
        System.out.println("Enter Booking ID to cancel:");
        int bookingId = scanner.nextInt();
        try {
            PreparedStatement preparedStatement = connection.prepareStatement("DELETE FROM booking WHERE id = ? AND user_id = ?");
            preparedStatement.setInt(1, bookingId);
            preparedStatement.setInt(2, currentUserId); // Ensure only the user's booking is canceled
            int rowsAffected = preparedStatement.executeUpdate();
            if (rowsAffected > 0) {
                System.out.println("Booking canceled successfully!");
            } else {
                System.out.println("Booking cancellation failed. Please check the Booking ID.");
            }
        } catch (SQLException e) {
            System.err.println("Error: " + e.getMessage());
        }
    }

}
