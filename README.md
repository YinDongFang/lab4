# lab4
## 1. 完善Customer类
整体代码类似`Supplier`类，代码复制过来修改就可以，不同的地方在于`AdminMainScreen`中`Customer`需要显示创建日期，不需要产品数量，所以不用要`directory`属性，多一个`created`属性。
```java
/*
 * To change this license header, choose License Headers in Project Properties.
 * To change this template file, choose Tools | Templates
 * and open the template in the editor.
 */
package Business.Users;

import Business.Abstract.User;
import java.util.Date;

/**
 *
 * @author AEDSpring2019
 */
public class Customer extends User implements Comparable<Customer>{
    
    private Date created;
    
    public Customer(String password, String userName) {
        super(password, userName, "CUSTOMER");
        created = new Date();
    }

    public Date getCreated() {
        return created;
    }

    @Override
    public int compareTo(Customer o) {
        return o.getUserName().compareTo(this.getUserName());
    }

    @Override
    public String toString() {
        return getUserName(); //To change body of generated methods, choose Tools | Templates.
    }
    
    public boolean verify(String password){
        return password.equals(getPassword());
    }
}

```
## 2. 填充`Customer`表格
作业要求完善`AdminMainScreen`中用户表格数据，参考供应商表格的方法即可。

完整的`population`代码
```java
    public void populate(){
        DefaultTableModel dtm = (DefaultTableModel)tableSup.getModel();
        dtm.setRowCount(0);
        for(User u : admin.getSuppDir().getSupplierList()){
            Supplier s = (Supplier)u;
            Object[] row = new Object[dtm.getColumnCount()];
            row[0]=s;
            row[1]=s.getDirectory().getProductList().size();
            dtm.addRow(row);
        }
        
        // populate the customer table
        dtm = (DefaultTableModel)tableCust.getModel();
        dtm.setRowCount(0);
        for(User u : admin.getCustDir().getCustomerList()){
            Customer c = (Customer)u;
            Object[] row = new Object[dtm.getColumnCount()];
            row[0]=c;
            row[1]=c.getCreated();
            dtm.addRow(row);
        }
    }
```
## 3. 完成角色创建
角色创建代码主要在`AdminCreateScreen.btnCreateActionPerformed`方法中实现，主要步骤有：
  
    1. 输入校验（正则表达式，两次密码是否相同）
    2. 根据选择的role创建不同的角色
    3. 添加到admin对应的用户字典中
    4. 显示成功界面

  完整的`btnCreateActionPerformed`代码
```java
    private void btnCreateActionPerformed(java.awt.event.ActionEvent evt) {                                          
        // 1. initialize components color
        jLabel1.setForeground(Color.black);
        jLabel2.setForeground(Color.black);
        jLabel3.setForeground(Color.black);
        txtUser.setForeground(Color.black);
        txtPword.setForeground(Color.black);
        txtRePword.setForeground(Color.black);
        radioCustomer.setForeground(Color.black);
        radioSupplier.setForeground(Color.black);
        // 2. validate user name & password
        // 2.1 validate user name by Pattern & Matcher
        String username = txtUser.getText();
        Pattern regex = Pattern.compile("^[a-z0-9A-Z]+_[a-z0-9A-Z]+@[a-z0-9A-Z]+\\.[a-z0-9A-Z]+$");
        Matcher matcher = regex.matcher(username);
        boolean isMatched = matcher.matches();
        if (!isMatched) {
            // if not matched, change the color and show message dialog.
            jLabel1.setForeground(Color.red);
            txtUser.setForeground(Color.red);
            JOptionPane.showMessageDialog(null, "Please enter correct email");
            return;
        }
        // 2.2 validate password by Pattern & Matcher
        String pwd = txtPword.getText();
        regex = Pattern.compile("^(?![A-Za-z0-9]+$)(?![a-z0-9\\W]+$)(?![A-Za-z\\W]+$)(?![A-Z0-9\\W]+$)[a-zA-Z0-9\\$\\*#&]{6,}$");
        matcher = regex.matcher(pwd);
        isMatched = matcher.matches();
        if (!isMatched) {
            // if not matched, change the color and show message dialog.
            jLabel2.setForeground(Color.red);
            txtPword.setForeground(Color.red);
            JOptionPane.showMessageDialog(null, "Please enter correct password");
            return;
        }
        // 2.3 validate re-enter password
        String pwd2 = txtRePword.getText();
        isMatched = pwd2.equals(pwd);
        if (!isMatched) {
            // if not matched, change the color and show message dialog.
            jLabel3.setForeground(Color.red);
            txtRePword.setForeground(Color.red);
            JOptionPane.showMessageDialog(null, "Please re-enter correct password");
            return;
        }
        // 2.4 validate user role
        if (!radioCustomer.isSelected() && !radioSupplier.isSelected()) {
            radioCustomer.setForeground(Color.red);
            radioSupplier.setForeground(Color.red);
            JOptionPane.showMessageDialog(null, "Please check user role");
            return;
        }
        // 3. create user & add user to admin's directory
        User user;
        if (radioCustomer.isSelected()) {
            user = new Customer(pwd, username);
            admin.getCustDir().getCustomerList().add(user);
        } else {
            user = new Supplier(pwd, username);
            admin.getSuppDir().getSupplierList().add(user);
        }
        JOptionPane.showMessageDialog(null, "Successful!");
        // 4. show the SuccessScreen
        CardLayout layout = (CardLayout)panelRight.getLayout();
        panelRight.remove(this);
        panelRight.add(new SuccessScreen(user));
        layout.next(panelRight);
    }    
```
## 4. 完成登录
用户和供应商登录用的是同一个界面，区别的方法是根据传入的用户字典中用户的身份。

这里考虑到如果传入的用户列表是空的处理，可以弹出提示创建角色并且禁用按钮，避免用户仍然进行操作产生错误。这部分操作在`initialize`中完成。

因为传入的用户列表是`Costomer`或者`Supplier`，不用做区分，根据列表中第一个用户的`role`做判定就可以。

完整的`initialize`代码：
```java
    private void initialize() {
        txtTitle.setText("Unknown Login Screen");
        comboUser.removeAllItems();
        // handle empty list
        if (list.size() == 0) {
            // show message dialog
            JOptionPane.showMessageDialog(null, "Please create user");
            btnSubmit.setEnabled(false);
            return;
        }
        //text should either be "Supplier Login Screen" OR "Customer Login Screen"
        //Based on the selection
        String role = list.get(0).getRole();
        if (role.equals(new Customer("", "").getRole())) {
            txtTitle.setText("Customer Login Screen");
        } else {
            txtTitle.setText("Supplier Login Screen");
        }
        //only customer or suppliers should be listed based on the selection
        for (User user : list) {
            comboUser.addItem(user);
        }
        // set first user as default selected user
        comboUser.setSelectedIndex(0);
    }
```

登录时只需要对密码进行校验就可以。`initialize`中设置了默认选中第一个角色，所以这里不用对角色进行是否为空的判定。

完整的`btnSubmitActionPerformed`代码：
```java
    private void btnSubmitActionPerformed(java.awt.event.ActionEvent evt) {                                          
        // verify password
        User user = (User) comboUser.getSelectedItem();
        String pwd = txtPword.getText();
        boolean correct = user.verify(pwd);
        if (correct) {
            // show SuccessScreen
            CardLayout layout = (CardLayout) panelRight.getLayout();
            panelRight.remove(this);
            panelRight.add(new SuccessScreen(user));
            layout.next(panelRight);
        } else {
            // show message
            JOptionPane.showMessageDialog(null, "Please enter correct password");
        }
    }   
```