# Kiosk

	package idox;

	import javax.swing.*;

	import javax.swing.border.EmptyBorder;

	import java.awt.*;

	import java.awt.event.*;

	import java.util.ArrayList;

	import java.util.HashMap;

	import java.util.Map;

	public class swingUI extends JFrame {

	private static final long serialVersionUID = 1L;

	private JPanel contentPane;

	private JLabel orderLabel;

	private JLabel priceLabel;

	private ArrayList<String> orders = new ArrayList<>();

	private int totalPrice = 0;

	private int orderNumber = 1; // 주문 번호

	private CardLayout cardLayout;

	private JPanel mainPanel;

	private JLabel orderCompletionLabel;

	private JLabel totalCompletionLabel;

	private JLabel orderNumberLabel; // 주문 번호 표시

	private final Map<String, Integer> inventory = new HashMap<>();

	private final Map<String, Integer> sales = new HashMap<>(); // 판매된 커피의 수량

	private final Map<String, Integer> revenue = new HashMap<>(); // 총 매출

	// 비밀번호 설정

	private static final String ADMIN_PASSWORD = "shinil";

	public static void main(String[] args) {

		EventQueue.invokeLater(() -> {

			try {

				swingUI frame = new swingUI();

				frame.setVisible(true);

			} catch (Exception e) {

				e.printStackTrace();

			}

		});

	}

	public swingUI() {

		setDefaultCloseOperation(JFrame.EXIT_ON_CLOSE);

		setBounds(100, 100, 1950, 1050);

		contentPane = new JPanel();

		contentPane.setBorder(new EmptyBorder(10, 10, 10, 10));

		setContentPane(contentPane);

		contentPane.setLayout(new BorderLayout());

		cardLayout = new CardLayout();

		mainPanel = new JPanel(cardLayout);

		contentPane.add(mainPanel, BorderLayout.CENTER);

		// 초기 재고 설정

		inventory.put("에스프레소", 80);

		inventory.put("라떼", 70);

		inventory.put("카푸치노", 50);

		sales.put("에스프레소", 0);

		sales.put("라떼", 0);

		sales.put("카푸치노", 0);

		revenue.put("에스프레소", 0);

		revenue.put("라떼", 0);

		revenue.put("카푸치노", 0);

		createLanguageSelectionPanel();

		createCoffeeSelectionPanel();

		createOrderCompletionPanel();

	}

	private void createLanguageSelectionPanel() {

		JPanel startPanel = new JPanel();

		startPanel.setLayout(null);

		JButton eatInButton = new JButton("먹고가기");

		eatInButton.setBounds(500, 750, 400, 175);

		eatInButton.addActionListener(e -> switchToCoffeeSelection());

		JButton takeOutButton = new JButton("포장하기");

		takeOutButton.setBounds(1100, 750, 400, 175);

		takeOutButton.addActionListener(e -> switchToCoffeeSelection());

		startPanel.add(eatInButton);

		startPanel.add(takeOutButton);

		mainPanel.add(startPanel, "LanguageSelection");

		cardLayout.show(mainPanel, "LanguageSelection");

	}

	private void createCoffeeSelectionPanel() {

		JPanel coffeePanel = new JPanel();

		coffeePanel.setLayout(null);

		orderLabel = new JLabel("주문: 없음");

		orderLabel.setBounds(20, 20, 1000, 100);

		priceLabel = new JLabel("총 가격: 0원");

		priceLabel.setBounds(20, 20, 300, 30);

		coffeePanel.add(orderLabel);

		coffeePanel.add(priceLabel);

		// 각 커피별 버튼 추가

		addCoffeeButton(coffeePanel, "에스프레소", 3000, 50, 100);

		addCoffeeButton(coffeePanel, "라떼", 4500, 50, 160);

		addCoffeeButton(coffeePanel, "카푸치노", 4000, 50, 220);

		JButton completeOrderButton = new JButton("주문 완료");

		completeOrderButton.setBounds(50, 300, 200, 50);

		completeOrderButton.addActionListener(e -> switchToOrderCompletion());

		coffeePanel.add(completeOrderButton);

		JButton backButton = new JButton("처음으로 돌아가기");

		backButton.setBounds(300, 300, 200, 50);

		backButton.addActionListener(e -> switchToLanguageSelection());

		coffeePanel.add(backButton);

		// 초기화 버튼 추가

		JButton resetOrderButton = new JButton("주문 초기화");

		resetOrderButton.setBounds(50, 380, 200, 50);

		resetOrderButton.addActionListener(e -> resetOrder());

		coffeePanel.add(resetOrderButton);

		// 관리자 버튼 추가

		JButton adminButton = new JButton("관리자 패널");

		adminButton.setBounds(1780, 960, 120, 35);

		adminButton.addActionListener(e -> showAdminLoginDialog());

		coffeePanel.add(adminButton);

		mainPanel.add(coffeePanel, "CoffeeSelection");

	}

	private void addCoffeeButton(JPanel panel, String coffeeType, int price, int x, int y) {

		JLabel stockLabel = new JLabel(coffeeType + " 재고: " + inventory.get(coffeeType));

		stockLabel.setBounds(x + 230, y, 200, 30); // 재고 위치와 크기

		JButton button = new JButton(coffeeType + " - " + price + "원");

		button.setBounds(x, y, 200, 50);

		button.addActionListener(e -> openCoffeeOptions(coffeeType, price, button, stockLabel));

		panel.add(button);

		panel.add(stockLabel);

	}

	private void createOrderCompletionPanel() {

		JPanel orderPanel = new JPanel();

		orderPanel.setLayout(null);

		orderCompletionLabel = new JLabel("주문 내역: ");

		orderCompletionLabel.setBounds(50, 20, 500, 30);

		totalCompletionLabel = new JLabel("총 가격: 0원");

		totalCompletionLabel.setBounds(50, 60, 500, 30);

		orderNumberLabel = new JLabel("주문 번호: 1");

		orderNumberLabel.setBounds(50, 100, 300, 30);

		orderPanel.add(orderNumberLabel);

		orderPanel.add(orderCompletionLabel);

		orderPanel.add(totalCompletionLabel);

		JButton paymentButton = new JButton("결제하기");

		paymentButton.setBounds(50, 200, 200, 50);

		paymentButton.addActionListener(e -> completePayment());

		orderPanel.add(paymentButton);

		JButton backButton = new JButton("처음으로 돌아가기");

		backButton.setBounds(300, 200, 200, 50);

		backButton.addActionListener(e -> switchToLanguageSelection());

		orderPanel.add(backButton);

		mainPanel.add(orderPanel, "OrderCompletion");

	}

	private void openCoffeeOptions(String coffeeType, int price, JButton coffeeButton, JLabel stockLabel) {

		CoffeeOptionPanel optionPanel = new CoffeeOptionPanel(coffeeType, price, coffeeButton, stockLabel);

		mainPanel.add(optionPanel, "CoffeeOption");

		cardLayout.show(mainPanel, "CoffeeOption");

	}

	private void switchToLanguageSelection() {

		cardLayout.show(mainPanel, "LanguageSelection");

	}

	private void switchToCoffeeSelection() {

		cardLayout.show(mainPanel, "CoffeeSelection");

	}

	private void switchToOrderCompletion() {

		orderCompletionLabel.setText("주문 내역: " + String.join(", ", orders));

		totalCompletionLabel.setText("총 가격: " + totalPrice + "원");

		orderNumberLabel.setText("주문 번호: " + orderNumber);

		cardLayout.show(mainPanel, "OrderCompletion");

	}

	private void addOrder(String coffeeType, int price, JButton button, JLabel stockLabel, String options) {

		if (orders.size() < 7) {

			if (inventory.get(coffeeType) > 0) {

				orders.add(coffeeType + " " + options);

				totalPrice += price;

				orderLabel.setText("주문: " + orders);

				priceLabel.setText("총 가격: " + totalPrice + "원");

				inventory.put(coffeeType, inventory.get(coffeeType) - 1);

				stockLabel.setText(coffeeType + " 재고: " + inventory.get(coffeeType));

				button.setEnabled(inventory.get(coffeeType) > 0);

				sales.put(coffeeType, sales.get(coffeeType) + 1);

				revenue.put(coffeeType, revenue.get(coffeeType) + price);

			} else {

				JOptionPane.showMessageDialog(this, coffeeType + "의 재고가 부족합니다.", "오류", JOptionPane.ERROR_MESSAGE);

			}

		} else {

			JOptionPane.showMessageDialog(this, "최대 7개의 커피만 주문할 수 있습니다.", "오류", JOptionPane.ERROR_MESSAGE);

		}

	}

	private void completePayment() {

		if (orders.isEmpty()) {

			JOptionPane.showMessageDialog(this, "주문이 없습니다.", "오류", JOptionPane.ERROR_MESSAGE);

		} else {

			ReceiptFrame receiptFrame = new ReceiptFrame(orders, totalPrice, orderNumber);

			receiptFrame.setVisible(true);

			orderNumber++;

			resetOrder();

			switchToCoffeeSelection();

		}

	}

	private void resetOrder() {

		orders.clear();

		totalPrice = 0;

		orderLabel.setText("주문: 없음");

		priceLabel.setText("총 가격: 0원");

		JOptionPane.showMessageDialog(this, "결제 완료. 감사합니다!", "초기화", JOptionPane.INFORMATION_MESSAGE);

		updateButtonStates();

	}

	private void updateButtonStates() {

		for (Component comp : mainPanel.getComponents()) {

			if (comp instanceof JPanel) {

				for (Component button : ((JPanel) comp).getComponents()) {

					if (button instanceof JButton) {

						String buttonText = ((JButton) button).getText();

						String coffeeType = buttonText.split(" - ")[0];

						button.setEnabled(inventory.get(coffeeType) > 0);

					}

				}

			}

		}

	}

	// 비밀번호 입력 다이얼로그 메소드

	private void showAdminLoginDialog() {

		JPasswordField passwordField = new JPasswordField(10);

		JPanel panel = new JPanel();

		panel.add(new JLabel("관리자 비밀번호 입력:"));

		panel.add(passwordField);

		int option = JOptionPane.showConfirmDialog(this, panel, "관리자 로그인", JOptionPane.OK_CANCEL_OPTION,

				JOptionPane.PLAIN_MESSAGE);

		if (option == JOptionPane.OK_OPTION) {

			String enteredPassword = new String(passwordField.getPassword());

			if (ADMIN_PASSWORD.equals(enteredPassword)) {

				showAdminPanel();

			} else {

				JOptionPane.showMessageDialog(this, "잘못된 비밀번호입니다.", "오류", JOptionPane.ERROR_MESSAGE);

			}

		}

	}

	private void showSalesStatistics() {

		// TODO Auto-generated method stub

	}

	// 비밀번호 확인 메소드

	private boolean checkPassword(String enteredPassword) {

		return ADMIN_PASSWORD.equals(enteredPassword);

	}

	// 관리자 패널에서 메뉴와 옵션을 비활성화 할 수 있는 기능 추가

	public class CoffeeOptionPanel extends JPanel {

		private String coffeeType;

		private int basePrice;

		private JButton coffeeButton;

		private JLabel stockLabel;

		private JCheckBox extraShotCheckBox;

		private JCheckBox extraMilkCheckBox;

		private JCheckBox extraLightCheckBox;

		private JCheckBox extraWeightCheckBox;

		private JCheckBox extraZeroCheckBox;

		private JCheckBox extraHoneyCheckBox;

		private JCheckBox extraCreamCheckBox;

		public CoffeeOptionPanel(String coffeeType, int basePrice, JButton coffeeButton, JLabel stockLabel) {

			this.coffeeType = coffeeType;

			this.basePrice = basePrice;

			this.coffeeButton = coffeeButton;

			this.stockLabel = stockLabel;

			setLayout(new BoxLayout(this, BoxLayout.Y_AXIS));

			// 옵션 체크박스 생성

			extraShotCheckBox = new JCheckBox("샷 추가 (+300원)");

			extraMilkCheckBox = new JCheckBox("우유 많이 (+200원)");

			extraLightCheckBox = new JCheckBox("연하게");

			extraWeightCheckBox = new JCheckBox("진하게");

			extraZeroCheckBox = new JCheckBox("제로/저지방");

			extraHoneyCheckBox = new JCheckBox("꿀 추가 (+300원)");

			extraCreamCheckBox = new JCheckBox("휘핑 추가 (+300원)");

			add(extraShotCheckBox);

			add(extraMilkCheckBox);

			add(extraLightCheckBox);

			add(extraWeightCheckBox);

			add(extraZeroCheckBox);

			add(extraHoneyCheckBox);

			add(extraCreamCheckBox);

			JButton confirmButton = new JButton("확인");

			confirmButton.addActionListener(e -> confirmOptions());

			add(confirmButton);

			JButton backButton = new JButton("뒤로 가기");

			backButton.addActionListener(e -> backToCoffeeSelection());

			add(backButton);

		}

		private void confirmOptions() {

			int price = basePrice;

			StringBuilder options = new StringBuilder(); // StringBuilder로 옵션을 연결

			if (extraShotCheckBox.isSelected()) {

				price += 300;

				options.append("샷 추가, ");

			}

			if (extraMilkCheckBox.isSelected()) {

				price += 200;

				options.append("우유 많이, ");

			}

			if (extraLightCheckBox.isSelected()) {

				options.append("연하게, ");

			}

			if (extraWeightCheckBox.isSelected()) {

				options.append("진하게, ");

			}

			if (extraZeroCheckBox.isSelected()) {

				options.append("제로/저지방, ");

			}

			if (extraHoneyCheckBox.isSelected()) {

				price += 300;

				options.append("꿀 추가, ");

			}

			if (extraCreamCheckBox.isSelected()) {

				price += 300;

				options.append("휘핑 추가, ");

			}

			if (options.length() > 0) {

				options.delete(options.length() - 2, options.length());

			}

			if (options.length() == 0) {

				options.append("(옵션 : 없음)");

			} else {

				options.insert(0, "(옵션 : ");

				options.append(")");

			}

			addOrder(coffeeType, price, coffeeButton, stockLabel, options.toString());

			switchToCoffeeSelection();

		}

		private void backToCoffeeSelection() {

			switchToCoffeeSelection();

		}

	}

	private void showAdminPanel() {

		// 관리자 패널을 위한 패널 생성

		JPanel adminPanel = new JPanel();

		adminPanel.setLayout(null);

		adminPanel.setPreferredSize(new Dimension(1000, 500)); // 해상도에 맞는 크기 설정

		// 메뉴별 재고 입력 필드 및 버튼 추가

		addMenuStockField(adminPanel, "에스프레소", 50);

		addMenuStockField(adminPanel, "라떼", 120);

		addMenuStockField(adminPanel, "카푸치노", 190);

		// 메뉴 활성화/비활성화 버튼들

		JButton espressoButton = new JButton("에스프레소 " + (inventory.get("에스프레소") > 0 ? "활성" : "비활성"));

		espressoButton.setBounds(50, 300, 250, 50);

		espressoButton.addActionListener(e -> toggleMenu("에스프레소", espressoButton));

		adminPanel.add(espressoButton);

		JButton latteButton = new JButton("라떼 " + (inventory.get("라떼") > 0 ? "활성" : "비활성"));

		latteButton.setBounds(50, 370, 250, 50);

		latteButton.addActionListener(e -> toggleMenu("라떼", latteButton));

		adminPanel.add(latteButton);

		JButton cappuccinoButton = new JButton("카푸치노 " + (inventory.get("카푸치노") > 0 ? "활성" : "비활성"));

		cappuccinoButton.setBounds(50, 440, 250, 50);

		cappuccinoButton.addActionListener(e -> toggleMenu("카푸치노", cappuccinoButton));

		adminPanel.add(cappuccinoButton);

		// 판매 통계 영역

		JTextArea salesReportArea = new JTextArea();

		salesReportArea.setEditable(false);

		salesReportArea.setBounds(350, 50, 1500, 600);

		StringBuilder salesReport = new StringBuilder("커피 판매 현황:\n");

		for (String coffeeType : inventory.keySet()) {

			salesReport.append(coffeeType)

					.append(" - 판매 수량: ").append(sales.get(coffeeType))

					.append(", 총 매출: ").append(revenue.get(coffeeType)).append("원\n");

		}

		salesReportArea.setText(salesReport.toString());

		adminPanel.add(salesReportArea);

		// 총 매출 표시

		JLabel totalRevenueLabel = new JLabel("총 매출: " + getTotalRevenue() + "원");

		totalRevenueLabel.setBounds(350, 670, 500, 50);

		adminPanel.add(totalRevenueLabel);

		// 패널에 추가하고 관리자 패널을 표시

		JOptionPane.showMessageDialog(this, adminPanel, "관리자 패널", JOptionPane.PLAIN_MESSAGE);

	}

	private int getTotalRevenue() {

		int totalRevenue = 0;

		// 커피 유형별로 매출을 합산

		for (String coffeeType : revenue.keySet()) {

			totalRevenue += revenue.get(coffeeType); // 커피별 매출 합산

		}

		return totalRevenue; // 총 매출 반환

	}

	private void addMenuStockField(JPanel panel, String coffeeType, int yPosition) {

		// 재고를 입력할 텍스트 필드 추가

		JLabel stockLabel = new JLabel(coffeeType + " 재고:");

		stockLabel.setBounds(50, yPosition, 100, 30);

		panel.add(stockLabel);

		JTextField stockField = new JTextField(String.valueOf(inventory.get(coffeeType))); // 현재 재고로 초기화

		stockField.setBounds(150, yPosition, 50, 30);

		panel.add(stockField);

		// 재고 변경 버튼 추가

		JButton updateButton = new JButton("업데이트");

		updateButton.setBounds(220, yPosition, 100, 30);

		updateButton.addActionListener(e -> updateStock(coffeeType, stockField));

		panel.add(updateButton);

	}

	private void updateStock(String coffeeType, JTextField stockField) {

		try {

			int newStock = Integer.parseInt(stockField.getText()); // 새로운 재고 값 읽기

			if (newStock >= 0) { // 재고는 음수일 수 없으므로 체크

				inventory.put(coffeeType, newStock); // 새로운 재고 값으로 업데이트

				JOptionPane.showMessageDialog(this, coffeeType + "의 재고가 " + newStock + "개로 변경되었습니다.");

			} else {

				JOptionPane.showMessageDialog(this, "재고는 음수일 수 없습니다.", "오류", JOptionPane.ERROR_MESSAGE);

			}

		} catch (NumberFormatException ex) {

			JOptionPane.showMessageDialog(this, "유효한 숫자를 입력하세요.", "오류", JOptionPane.ERROR_MESSAGE);

		}

	}

	private void toggleMenu(String coffeeType, JButton button) {

		boolean isActive = inventory.get(coffeeType) > 0;

		if (isActive) {

			// 비활성화 처리

			inventory.put(coffeeType, 0);

			button.setEnabled(false);

			button.setText(coffeeType + " 비활성");

		} else {

			// 활성화 처리

			inventory.put(coffeeType, 80);

			button.setEnabled(true);

			button.setText(coffeeType + " 활성");

		}

	}

	public static class ReceiptFrame extends JFrame {

		private JLabel receiptLabel;

		private JLabel totalLabel;

		public ReceiptFrame(ArrayList<String> orders, int totalPrice, int orderNumber) {

			setTitle("주문 영수증");

			setSize(300, 200);

			setDefaultCloseOperation(JFrame.DISPOSE_ON_CLOSE);

			setLocationRelativeTo(null);

			JPanel panel = new JPanel();

			panel.setLayout(new BoxLayout(panel, BoxLayout.Y_AXIS));

			receiptLabel = new JLabel("주문 내역: " + String.join(", ", orders));

			totalLabel = new JLabel("총 가격: " + totalPrice + "원");

			panel.add(new JLabel("주문 번호: " + orderNumber));

			panel.add(receiptLabel);

			panel.add(totalLabel);

			add(panel);

			}
		}
	}
