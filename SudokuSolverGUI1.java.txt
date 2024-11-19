import javax.swing.*;
import java.awt.*;
import java.awt.event.*;

public class SudokuSolverGUI1 extends JFrame {
    private static final int SIZE = 9;
    private JTextField[][] cells = new JTextField[SIZE][SIZE];
    private JButton solveButton, clearButton;

    public SudokuSolverGUI() {
        setTitle("Sudoku Solver");
        setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);
        setSize(550, 600);
        setLayout(new BorderLayout());

        // Create the Sudoku grid panel
        JPanel gridPanel = new JPanel(new GridLayout(SIZE, SIZE));
        gridPanel.setBorder(BorderFactory.createEmptyBorder(10, 10, 10, 10));

        for (int row = 0; row < SIZE; row++) {
            for (int col = 0; col < SIZE; col++) {
                cells[row][col] = new JTextField();
                cells[row][col].setHorizontalAlignment(JTextField.CENTER);
                cells[row][col].setFont(new Font("Arial", Font.BOLD, 20));

                // Limit to one character per cell and handle input validation
                cells[row][col].setDocument(new JTextFieldLimit(1));
                cells[row][col].addKeyListener(new CellKeyListener(row, col));

                // Set background color for alternating 3x3 subgrids
                boolean isShaded = ((row / 3) + (col / 3)) % 2 == 0;
                cells[row][col].setBackground(isShaded ? new Color(220, 220, 220) : Color.WHITE);

                // Thicker borders to separate 3x3 subgrids
                int top = row % 3 == 0 ? 4 : 1;
                int left = col % 3 == 0 ? 4 : 1;
                int bottom = row == SIZE - 1 ? 4 : 1;
                int right = col == SIZE - 1 ? 4 : 1;
                cells[row][col].setBorder(BorderFactory.createMatteBorder(top, left, bottom, right, Color.BLACK));

                gridPanel.add(cells[row][col]);
            }
        }

        // Solve and Clear buttons
        solveButton = new JButton("Solve Sudoku");
        solveButton.setFont(new Font("Arial", Font.BOLD, 16));
        solveButton.addActionListener(new SolveButtonListener());

        clearButton = new JButton("Clear Board");
        clearButton.setFont(new Font("Arial", Font.BOLD, 16));
        clearButton.addActionListener(e -> clearBoard());

        JPanel buttonPanel = new JPanel();
        buttonPanel.add(solveButton);
        buttonPanel.add(clearButton);

        add(gridPanel, BorderLayout.CENTER);
        add(buttonPanel, BorderLayout.SOUTH);
    }

    // Clears all cells on the board
    private void clearBoard() {
        for (int row = 0; row < SIZE; row++) {
            for (int col = 0; col < SIZE; col++) {
                cells[row][col].setText("");
                cells[row][col].setEditable(true);
            }
        }
    }

    private class SolveButtonListener implements ActionListener {
        public void actionPerformed(ActionEvent e) {
            int[][] grid = new int[SIZE][SIZE];

            // Populate the grid with values from the text fields, checking for errors
            try {
                for (int row = 0; row < SIZE; row++) {
                    for (int col = 0; col < SIZE; col++) {
                        String text = cells[row][col].getText();
                        if (!text.isEmpty()) {
                            int value = Integer.parseInt(text);
                            if (value < 1 || value > 9 || !isSafe(grid, row, col, value)) {
                                throw new IllegalArgumentException("Invalid Sudoku board configuration.");
                            }
                            grid[row][col] = value;
                        } else {
                            grid[row][col] = 0;
                        }
                    }
                }

                // Solve the Sudoku
                if (solveSudoku(grid)) {
                    // Update the grid with the solved values
                    for (int row = 0; row < SIZE; row++) {
                        for (int col = 0; col < SIZE; col++) {
                            cells[row][col].setText(String.valueOf(grid[row][col]));
                            cells[row][col].setEditable(false);
                        }
                    }
                } else {
                    JOptionPane.showMessageDialog(null, "No solution exists for the provided Sudoku puzzle.", "Error", JOptionPane.ERROR_MESSAGE);
                }
            } catch (NumberFormatException | IllegalArgumentException ex) {
                JOptionPane.showMessageDialog(null, "Please enter valid numbers (1-9) without conflicts.", "Error", JOptionPane.ERROR_MESSAGE);
            }
        }
    }

    // Helper function to solve Sudoku using backtracking
    private boolean solveSudoku(int[][] grid) {
        for (int row = 0; row < SIZE; row++) {
            for (int col = 0; col < SIZE; col++) {
                if (grid[row][col] == 0) {
                    for (int num = 1; num <= SIZE; num++) {
                        if (isSafe(grid, row, col, num)) {
                            grid[row][col] = num;
                            if (solveSudoku(grid)) return true;
                            grid[row][col] = 0;
                        }
                    }
                    return false;
                }
            }
        }
        return true;
    }

    // Helper function to check if a number can be placed in a cell
    private boolean isSafe(int[][] grid, int row, int col, int num) {
        for (int i = 0; i < SIZE; i++) {
            if (grid[row][i] == num || grid[i][col] == num) return false;
        }

        int startRow = row - row % 3;
        int startCol = col - col % 3;

        for (int i = 0; i < 3; i++) {
            for (int j = 0; j < 3; j++) {
                if (grid[i + startRow][j + startCol] == num) return false;
            }
        }
        return true;
    }

    // Custom document filter to limit JTextField to 1 character
    private static class JTextFieldLimit extends javax.swing.text.PlainDocument {
        private final int limit;

        JTextFieldLimit(int limit) {
            this.limit = limit;
        }

        public void insertString(int offset, String str, javax.swing.text.AttributeSet attr) throws javax.swing.text.BadLocationException {
            if (str == null) return;
            if ((getLength() + str.length()) <= limit && str.matches("[1-9]")) {
                super.insertString(offset, str, attr);
            } else {
                JOptionPane.showMessageDialog(null, "Only numbers 1-9 are allowed.", "Input Error", JOptionPane.WARNING_MESSAGE);
            }
        }
    }

    // Key listener to handle moving focus automatically
    private class CellKeyListener extends KeyAdapter {
        private final int row;
        private final int col;

        CellKeyListener(int row, int col) {
            this.row = row;
            this.col = col;
        }

        public void keyReleased(KeyEvent e) {
            if (cells[row][col].getText().length() == 1) {
                if (col < SIZE - 1) {
                    cells[row][col + 1].requestFocus();
                } else if (row < SIZE - 1) {
                    cells[row + 1][0].requestFocus();
                }
            }
        }
    }

    public static void main(String[] args) {
        SwingUtilities.invokeLater(() -> {
            SudokuSolverGUI frame = new SudokuSolverGUI();
            frame.setVisible(true);
        });
    }
}
