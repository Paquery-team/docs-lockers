La aplicación de administración de Lockers tiene sus usuarios en colección **users** de la base **clicknbox** y sus password están almacenados con hash bcrypt  
![][image1]  
Para cambiar un password, buscar el usuario por email con la query:  
![][image2]  
Generar el hash del nuevo password con la web app [https://appdevtools.com/bcrypt-generator](https://appdevtools.com/bcrypt-generator):   
![][image3]  
Copiar el hash y editar en el documento del usuario, el campo password:  
![][image4]  
Para guardar el cambio, debe hacer click en “UPDATE”  


[image1]: <./Images/ResetPassword/resetPassword1.png>

[image2]: <./Images/ResetPassword/resetPassword2.png>

[image3]: <./Images/ResetPassword/resetPassword3.png>

[image4]: <./Images/ResetPassword/resetPassword4.png>