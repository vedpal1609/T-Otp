# T-Otp


//=============================Controller ==================================

package com.totpmain.Controller;

import java.io.FileOutputStream;

import java.util.Base64;

import org.springframework.beans.factory.annotation.Autowired;

import org.springframework.web.bind.annotation.GetMapping;

import org.springframework.web.bind.annotation.PostMapping;

import org.springframework.web.bind.annotation.RequestParam;

import org.springframework.web.bind.annotation.ResponseBody;

import org.springframework.web.bind.annotation.RestController;

import dev.samstevens.totp.code.CodeVerifier;

import dev.samstevens.totp.exceptions.QrGenerationException;

import dev.samstevens.totp.qr.QrData;

import dev.samstevens.totp.qr.QrDataFactory;

import dev.samstevens.totp.qr.QrGenerator;

import dev.samstevens.totp.recovery.RecoveryCodeGenerator;

import dev.samstevens.totp.secret.SecretGenerator;


@RestController

public class Controller {

	
	@Autowired
    private SecretGenerator secretGenerator;

    @Autowired
    private QrDataFactory qrDataFactory;

    @Autowired
    private QrGenerator qrGenerator;
    @Autowired
    private RecoveryCodeGenerator recoveryCodeGenerator;
    
    @Autowired
    private CodeVerifier verifier;
 
    
    String secret ="";  

    @GetMapping("/QRGenerator")// --------QRGenerator --------
    
    public String setupDevice() throws QrGenerationException {
    	secret = secretGenerator.generate();       
        QrData data = qrDataFactory.newBuilder()
        		 .label("example@example.com")
                 .secret(secret)
                 .issuer("AppName")
                 .build();

        
        String qrCodeImage = getDataUriForImage(qrGenerator.generate(data), qrGenerator.getImageMimeType());
                            
	
			 
	    @PostMapping("/TokenVerify")  // ---------Token Verify
	    @ResponseBody
	    public String verify(@RequestParam String code) {
	         
			if (verifier.isValidCode( secret, code)) {
	            return "CORRECT CODE";
	        }
	        return "INCORRECT CODE";    
	        
	     
	    }
	 

	    @GetMapping("/QRrecovery")//------QR recovery
		 
	  public String[] recoveryCodes() {
	  String[] codes = recoveryCodeGenerator.generateCodes(16); return codes;
	  }
		  
}
    
       
    
//=================================== Configuration===================    
    
package com.totpmain.Configuration;

import java.net.UnknownHostException;

import org.springframework.context.annotation.Bean;

import org.springframework.context.annotation.Configuration;

import dev.samstevens.totp.code.HashingAlgorithm;

import dev.samstevens.totp.time.NtpTimeProvider;

import dev.samstevens.totp.time.TimeProvider;

@Configuration

public class AppConfig {
	
	@Bean
    public HashingAlgorithm hashingAlgorithm() {
        return HashingAlgorithm.SHA1;
    }
	
	
    @Bean
    public TimeProvider timeProvider() throws UnknownHostException {
        return new NtpTimeProvider("pool.ntp.org");
    }
}
    
    
    
//============== Properties  ===========

totp.secret.length=152

totp.code.length=8

totp.time.period=30

totp.time.discrepancy=4



//============Dependency============= 

                <dependency>
			<groupId>dev.samstevens.totp</groupId>
			<artifactId>totp</artifactId>
			<version>1.7.1</version>
		</dependency>
    
		<dependency>
			<groupId>commons-net</groupId>
			<artifactId>commons-net</artifactId>
			<version>3.6</version>
		</dependency>

    
    
    
