 @RequestMapping(value = "/login", method = RequestMethod.POST)
    public @ResponseBody
    JSONObject doLogin(@RequestBody(required =false)UserLogin userLogin, HttpServletRequest request) throws Exception {
//        log.info("LoginController > doLogin  >>>>> " + request.getAttribute("data"));

        UserLogin user = null;

//        Map<String, Object> jsonMap = (Map<String, Object>) request.getAttribute("data");
//        log.info("jsonMap " + jsonMap);
//        log.info("jsonMap " + jsonMap.get("userId"));
//
//        ////log.info("id " + userLogin.getUserId());
//        String userId = (String) jsonMap.get("userId");
        // FOR NEW FE Changes
        String userId = userLogin.getUserId();
//        String userId = CommonFunctions.getDcrypted(userLogin.getUserId());
//        log.info("userrid"+userId);
        userLogin.setUserId(userId);
        HttpSession session=request.getSession();

        session.setAttribute(CommonConstant.USER_ID, userId);
        session.setAttribute(CommonConstant.USER_SESSION_ID, session.getId() + "-" + session.getCreationTime());
        session.setAttribute(CommonConstant.USER_SESSION_ID, session.getId());




        user = loginService.doLogin(userLogin);
        String quarterFYearDate = loginService.getQuarterYear();
        String QFD[] = quarterFYearDate.split("~");
        String quarter = QFD[0];
        String financial_year = QFD[1];
        String quarter_end_date = QFD[2];
        String previousYearEndDate = QFD[3];
        String previousQuarterEndDate = QFD[4];
        user.setQuarterEndDate(quarter_end_date);
        user.setPreviousQuarterEndDate(previousQuarterEndDate);
        user.setPreviousYearEndDate(previousYearEndDate);
        user.setFinancialYear(financial_year);
        user.setQuarter(quarter);
//        log.info("user.getCircleCode()::"+user.getCircleCode());
        // Adding the Parameter to token for checking isCircle Authorized to SFTP Data

        boolean output=ifamsSftpService.getCirclesList(user.getCircleCode());
        log.info("Is Circle Exits for SFTP ::"+output);

        user.setIsCircleExist(String.valueOf(output));
        log.info("User getCircleExits :"+user.getIsCircleExist());




        if (!(("-1").equalsIgnoreCase(user.getIsUserExist()) || ("P").equalsIgnoreCase(user.getStatus()))) {
            user = loginService.getadditionalDetails(user);
            String token = CommonFunctions.getToken(user);
            session.setAttribute("TOKEN", token);
            int updated = loginService.saveToken(user, token);
            user.setToken(token);
            log.info("User save token :"+user.getToken());
        }

        String IV = AESGCM256.generateBase64IV();
        String SALT = AESGCM256.generateBase64Salt();

        //starting encryption
        user.setUserId(userId);
        user.setUserName(user.getUserName());
        user.setCircleCode(user.getCircleCode());
        user.setCircleName(user.getCircleName());
        user.setRole(user.getRole());
        user.setCapacity(user.getCapacity());
        if(!("P").equalsIgnoreCase(user.getStatus())){
        user.setStatus(user.getStatus());
        }
        user.setIsBranchFinal(user.getIsBranchFinal());
        user.setIsCircleFreeze(user.getIsCircleFreeze());
        user.setIsAuditorDig(user.getIsAuditorDig());
        user.setIsCheckerDig(user.getIsCheckerDig());
        user.setFrRMId("444444");
        user.setFrReportId("444444");
        user.setMocFlag(user.getMocFlag());
        user.setIsCircleExist(user.getIsCircleExist());

        ObjectMapper mapper = new ObjectMapper();
        String JsonString = mapper.writeValueAsString(user);

        String decryptedData = AESGCM256.encrypt(
                JsonString,
                IV,
                SALT
        );


        log.info("decryptedData::"+decryptedData);

        JSONObject jObj = new JSONObject();
        jObj.put("iv", IV);
        jObj.put("salt", SALT);
        jObj.put("user", decryptedData);
        return jObj;
    }
	
	
	 public static String encrypt(String plainText, String BASE64_IV, String BASE64_SALT) throws Exception {
        byte[] iv = Base64.getDecoder().decode(BASE64_IV);
        byte[] salt = Base64.getDecoder().decode(BASE64_SALT);
        byte[] passwordBytes =  BASE64_PASSWORD.getBytes(StandardCharsets.UTF_8);//Base64.getDecoder().decode(base64Password);

        SecretKeySpec key = deriveKey(passwordBytes, salt);

        Cipher cipher = Cipher.getInstance(CIPHER_TRANSFORMATION);
        GCMParameterSpec gcmSpec = new GCMParameterSpec(GCM_TAG_LENGTH * 8, iv);
        cipher.init(Cipher.ENCRYPT_MODE, key, gcmSpec);

        byte[] cipherText = cipher.doFinal(plainText.getBytes("UTF-8"));
        return Base64.getEncoder().encodeToString(cipherText);
    }

    public static String decrypt(String base64CipherText, String str_iv, String str_salt) throws Exception {
        byte[] iv = Base64.getDecoder().decode(str_iv);
        byte[] salt = Base64.getDecoder().decode(str_salt);
        byte[] encryptedBytes = Base64.getDecoder().decode(base64CipherText);
        byte[] passwordBytes = BASE64_PASSWORD.getBytes(StandardCharsets.UTF_8);

        SecretKeySpec key = deriveKey(passwordBytes, salt);

        Cipher cipher = Cipher.getInstance(CIPHER_TRANSFORMATION);
        GCMParameterSpec gcmSpec = new GCMParameterSpec(GCM_TAG_LENGTH * 8, iv);
        cipher.init(Cipher.DECRYPT_MODE, key, gcmSpec);

        byte[] decrypted = cipher.doFinal(encryptedBytes);
        return new String(decrypted, "UTF-8");
    }
////////////////////////////
	
	
	
	import { useState } from 'react';
import axios from 'axios';
import { encrypt } from '../../core/security/AES-GCM256';
const iv = crypto.getRandomValues(new Uint8Array(12)); // for encryption
const ivBase64 = btoa(String.fromCharCode.apply(null, iv)); // for be decryption
const salt = crypto.getRandomValues(new Uint8Array(16)); // for encryption
const saltBase64 = btoa(String.fromCharCode.apply(null, salt)); // for encryption

function isObjectEmpty(obj) {
  return Object.keys(obj).length === 0 && obj.constructor === Object;
}

const useApi = () => {
  const [data, setData] = useState(null);
  const [error, setError] = useState(null);
  const [loading, setLoading] = useState(false);

  const callApi = async (
    url,
    payload,
    method = 'POST',
    responseType = 'json',
    contentType = 'application/json',
    ...props
  ) => {
    setLoading(true);
    setError(null);
    try {
      if (contentType !== 'multipart/form-data') {
        if (payload) {
          const encryptedData = await encrypt(iv, salt, JSON.stringify(payload));
          payload = { iv: ivBase64, salt: saltBase64, data: encryptedData };
        }
      }

      let headers = {
        Authorization: `Bearer ${localStorage.getItem('token')}`,
        'Content-Type': contentType,
      };
      const config = {
        method,
        url: import.meta.env.VITE_BASE_SERVICE_URL + url,
        headers,
        responseType, // <-- key fix
        ...props,
      };
      if (method.toUpperCase() === 'GET') {
        config.params = payload || '';
      } else {
        config.data = payload || {};
      }

      const response = await axios(config);
      setData(response.data);
      return response.data;
    } catch (err) {
      console.error('API call error:', err);
      setError(err);
    } finally {
      setLoading(false);
    }
  };

  return { data, error, loading, callApi };
};

export default useApi;

/////////////////////////////////////

const passphrase = 'juVI+XqX90tQSqYPAmtVxg==';

function base64ToArrayBuffer(base64Text) {
  const binaryString = atob(base64Text);
  const len = binaryString.length;
  const bytes = new Uint8Array(len);
  for (let i = 0; i < len; i++) {
    bytes[i] = binaryString.charCodeAt(i);
  }
  return bytes.buffer;
}

async function encryptData(iv, salt, passphrase, data) {
  const encoder = new TextEncoder();
  const passphraseBuffer = encoder.encode(passphrase);
  const keyMaterial = await window.crypto.subtle.importKey('raw', passphraseBuffer, { name: 'PBKDF2' }, false, ['deriveKey']);

  const derivedKey = await window.crypto.subtle.deriveKey(
    {
      name: 'PBKDF2',
      salt: salt, //crypto.getRandomValues(new Uint8Array(16)),
      iterations: 1000,
      hash: 'SHA-256',
    },
    keyMaterial,
    {
      name: 'AES-GCM',
      length: 256,
    },
    true,
    ['encrypt', 'decrypt']
  );
  const encodedData = new TextEncoder().encode(data);
  const encryptedData = await window.crypto.subtle.encrypt({ name: 'AES-GCM', iv }, derivedKey, encodedData);

  return { iv, salt, encryptedData };
}

async function decryptData(iv, salt, passphrase, encryptedData) {
  const encoder = new TextEncoder();
  const passphraseBuffer = encoder.encode(passphrase);

  const encryptedArray = base64ToArrayBuffer(encryptedData);
  const keyMaterial = await window.crypto.subtle.importKey('raw', passphraseBuffer, { name: 'PBKDF2' }, false, ['deriveKey']);

  salt = base64ToArrayBuffer(salt);
  iv = base64ToArrayBuffer(iv);

  try {
    const derivedKey = await window.crypto.subtle.deriveKey(
      {
        name: 'PBKDF2',
        salt: salt, //crypto.getRandomValues(new Uint8Array(16)),
        iterations: 1000,
        hash: 'SHA-256',
      },
      keyMaterial,
      {
        name: 'AES-GCM',
        length: 256,
      },
      true,
      ['encrypt', 'decrypt']
    );

    //console.log('444444');

    const decryptedData = await window.crypto.subtle.decrypt({ name: 'AES-GCM', iv }, derivedKey, encryptedArray);
    //console.log('5555555');
    const dec = new TextDecoder();

    //return { iv, salt, dec.decode(decryptedData)};
    return dec.decode(decryptedData);
  } catch (e) {
    console.error(e);
  }
}

async function decrypt(iv, salt, data) {
 
  return decryptData(iv, salt, passphrase, data)
    .then((value) => {
     
      try {
        return value;
      } catch (e) {
        console.error('decrypt  ' + e);
      }
    })
    .catch(() => {});
}

function encrypt(iv, salt, data) {
  return encryptData(iv, salt, passphrase, data)
    .then((value) => {
      return btoa(String.fromCharCode.apply(null, new Uint8Array(value.encryptedData)));
    })
    .catch((e) => {
      console.error(e);
    });
}

export { encrypt, decrypt };

