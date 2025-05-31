# clientserver
1. Chat Server Code (ChatServer.java)
java
Copy
Edit
import java.io.*;
import java.net.*;
import java.util.*;

public class ChatServer {
    private static Set<PrintWriter> clientWriters = new HashSet<>();

    public static void main(String[] args) throws IOException {
        System.out.println("Chat server started...");
        ServerSocket serverSocket = new ServerSocket(5000);

        while (true) {
            Socket clientSocket = serverSocket.accept();
            System.out.println("New client connected.");
            new ClientHandler(clientSocket).start();
        }
    }

    private static class ClientHandler extends Thread {
        private Socket socket;
        private PrintWriter out;

        public ClientHandler(Socket socket) {
            this.socket = socket;
        }

        public void run() {
            try (
                BufferedReader in = new BufferedReader(
                    new InputStreamReader(socket.getInputStream()));
            ) {
                out = new PrintWriter(socket.getOutputStream(), true);
                synchronized (clientWriters) {
                    clientWriters.add(out);
                }

                String message;
                while ((message = in.readLine()) != null) {
                    System.out.println("Received: " + message);
                    synchronized (clientWriters) {
                        for (PrintWriter writer : clientWriters) {
                            writer.println(message);
                        }
                    }
                }
            } catch (IOException e) {
                System.out.println("Error: " + e.getMessage());
            } finally {
                try {
                    socket.close();
                } catch (IOException e) {}
                synchronized (clientWriters) {
                    clientWriters.remove(out);
                }
            }
        }
    }
}
 2. Chat Client Code (ChatClient.java)
java
Copy
Edit
import java.io.*;
import java.net.*;

public class ChatClient {
    public static void main(String[] args) throws IOException {
        Socket socket = new Socket("localhost", 5000);
        System.out.println("Connected to chat server.");

        new ReadThread(socket).start();
        new WriteThread(socket).start();
    }

    private static class ReadThread extends Thread {
        private BufferedReader reader;

        public ReadThread(Socket socket) {
            try {
                reader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
            } catch (IOException e) {
                System.out.println("ReadThread error: " + e.getMessage());
            }
        }

        public void run() {
            while (true) {
                try {
                    String message = reader.readLine();
                    if (message != null) {
                        System.out.println("\n" + message);
                    }
                } catch (IOException e) {
                    System.out.println("Read error: " + e.getMessage());
                    break;
                }
            }
        }
    }

    private static class WriteThread extends Thread {
        private PrintWriter writer;
        private BufferedReader consoleReader;

        public WriteThread(Socket socket) {
            try {
                writer = new PrintWriter(socket.getOutputStream(), true);
                consoleReader = new BufferedReader(new InputStreamReader(System.in));
            } catch (IOException e) {
                System.out.println("WriteThread error: " + e.getMessage());
            }
        }

        public void run() {
            while (true) {
                try {
                    String message = consoleReader.readLine();
                    writer.println(message);
                } catch (IOException e) {
                    System.out.println("Write error: " + e.getMessage());
                    break;
                }
            }
        }
    }
}
