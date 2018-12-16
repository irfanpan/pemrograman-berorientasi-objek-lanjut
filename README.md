# pemrograman-berorientasi-objek-lanjut
source code pemrograman berorientasi objek lanjut

- Implementasi TCP Socket(serverchat.java)

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.ServerSocket;
import java.net.Socket;
import java.util.HashSet;

public class ServerChat {

    private static final int PORT = 9001;
    private static HashSet<String> names = new HashSet<>();
    private static HashSet<PrintWriter> printWriters = new HashSet<>();

    public static void main(String[] args) throws IOException {
        System.out.println("server jalan pada port : " + PORT);
        ServerSocket serverSocket = new ServerSocket(PORT);
        try {
            while (true) {
                new Handler(serverSocket.accept()).start();
            }
        } catch (Exception e) {
            System.out.println(e);
        } finally {
            serverSocket.close();
        }
    }

    private static class Handler extends Thread {

        private String name;
        private Socket socket;
        private BufferedReader bufferedReader;
        private PrintWriter printWriter;

        public Handler(Socket socket) {
            this.socket = socket;
        }

        @Override
        public void run() {
            try {
                bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
                printWriter = new PrintWriter(socket.getOutputStream(), true);

                while (true) {
                    printWriter.println("submitname");
                    name = bufferedReader.readLine();

                    if (name == null) {
                        return;
                    }

                    synchronized (names) {
                    
                        if (!names.contains(name)) {
                            names.add(name);
                            break;
                        }
                    
                    }
                }

                printWriter.println("nameaccepted");
                printWriters.add(printWriter);

                while (true) {
                    String input = bufferedReader.readLine();
                    
                    if (input == null) {
                        return;
                    }
                    
                    printWriters.stream().forEach((pw) -> {
                        pw.println("message " + name + " : " + input);
                    });

                }

            } catch (IOException e) {
                System.out.println(e);
            } finally {

                if (name != null) {
                    names.remove(name);
                }
            
                if (printWriter != null) {
                    printWriters.remove(printWriter);
                }
            
                try {
                    socket.close();
                } catch (IOException e) {
                    System.out.println(e);
                }
            }
        }
    }
}

- membuat sebuah client(clientchat.java)

import java.awt.event.ActionEvent;
import java.awt.Dimension;
import java.awt.Toolkit;
import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.io.PrintWriter;
import java.net.Socket;
import javax.swing.JFrame;
import javax.swing.JOptionPane;
import javax.swing.JScrollPane;
import javax.swing.JTextArea;
import javax.swing.JTextField;

public class ClientChat {

    private BufferedReader bufferedReader;
    private PrintWriter printWriter;
    private JFrame jFrame = new JFrame("aplikasi chat");
    private JTextField jTextField = new JTextField(40);
    private JTextArea jTextArea = new JTextArea(8, 40);

    public ClientChat() {
        jTextField.setEditable(Boolean.FALSE);
        jTextArea.setEditable(Boolean.FALSE);
        jFrame.setSize(500, 500);
        jFrame.getContentPane().add(jTextField, "North");
        jFrame.getContentPane().add(new JScrollPane(jTextArea), "Center");
        Dimension dimension = Toolkit.getDefaultToolkit().getScreenSize();
        jFrame.setLocation((dimension.width / 2) - (jFrame.getSize().width / 2), (dimension.height / 2) - (jFrame.getSize().height / 2));

        jTextField.addActionListener((ActionEvent e) -> {
            printWriter.println(jTextField.getText());
            jTextField.setText("");
        });
    }

    public String getServerAddress() {
        return JOptionPane.showInputDialog(
            jFrame,
            "masukan ip address",
            "selamat datang di aplikasi chat",
            JOptionPane.QUESTION_MESSAGE
        );
    }

    public String getName() {
        return JOptionPane.showInputDialog(
            jFrame,
            "Masukkan nama anda",
            "selamat datang di aplikasi chat",
            JOptionPane.QUESTION_MESSAGE
        );
    }

    private void run() throws IOException {
        String serverAddress = getServerAddress();
        Socket socket = new Socket(serverAddress, 9001);
        bufferedReader = new BufferedReader(new InputStreamReader(socket.getInputStream()));
        printWriter = new PrintWriter(socket.getOutputStream(), true);

        while (true) {
            String line = bufferedReader.readLine();
            if (line.startsWith("submitname")) {
                printWriter.println(getName());
            } else if (line.startsWith("nameaccepted")) {
                jTextField.setEditable(Boolean.TRUE);
            } else if (line.startsWith("message")) {
                jTextArea.append(line.substring(8) + "\n");
            }
        }
    }

    public static void main(String[] args) throws IOException {
        ClientChat clientChat = new ClientChat();
        clientChat.jFrame.setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        clientChat.jFrame.setVisible(Boolean.TRUE);
        clientChat.run();
    }

}

- UDPserver.java

import java.io.IOException;
import java.net.DatagramPacket;
import java.net.DatagramSocket;

public class UDPServer {

    public static void main(String args[]) {
        DatagramSocket datagramSocket = null;

        try {
            datagramSocket = new DatagramSocket(8888);

            byte[] buffer = new byte[65536];
            DatagramPacket datagramPacket = new DatagramPacket(buffer, buffer.length);

            System.out.println("socket server jalan, menunggu data yang dikirim");

            while (true) {
                datagramSocket.receive(datagramPacket);
                byte[] data = datagramPacket.getData();
                String s = new String(data, 0, datagramPacket.getLength());

                System.out.println(datagramPacket.getAddress().getHostAddress() + " : " + datagramPacket.getPort() + " : " + s);

                s = "data yang terkirim : " + s;
                DatagramPacket packet = new DatagramPacket(s.getBytes(), s.getBytes().length, datagramPacket.getAddress(), datagramPacket.getPort());
                datagramSocket.send(packet);
            }

        } catch (IOException e) {
            System.out.println(e);
        }
    }
}

- UDPClient.java

import java.io.BufferedReader;
import java.io.IOException;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPClient {

    public static void main(String args[]) {
        DatagramSocket datagramSocket = null;
        String s;
        BufferedReader bufferedReader = new BufferedReader(new InputStreamReader(System.in));

        try {
            datagramSocket = new DatagramSocket();
            InetAddress inetAddress = InetAddress.getByName("127.0.0.1");

            while (true) {
                System.out.print("Masukkan pesan anda : ");
                s = bufferedReader.readLine();
                byte[] b = s.getBytes();

                DatagramPacket datagramPacket = new DatagramPacket(b, b.length, inetAddress, 8888);
                datagramSocket.send(datagramPacket);

                byte[] buffer = new byte[65536];
                DatagramPacket packet = new DatagramPacket(buffer, buffer.length);
                datagramSocket.receive(packet);

                byte[] data = packet.getData();
                s = new String(data, 0, packet.getLength());

                System.out.println(packet.getAddress().getHostName() + " : " + packet.getPort() + " : " + s);
            }

        } catch (IOException e) {
            System.out.println(e);
        }
    }
}
