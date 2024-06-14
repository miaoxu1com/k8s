const crypto = require('crypto');

// 辅助函数将整数数组转换为Buffer
function wordArrayToBuffer(wordArray) {
    const buffer = Buffer.alloc(wordArray.length * 4);
    for (let i = 0; i < wordArray.length; i++) {
        buffer.writeUInt32BE(wordArray[i], i * 4);
    }
    return buffer;
}

// 给定的密钥和IV
const keyWords = [1, 2, 3, 4]; // 这是之前定义的密钥，保持不变
const ivWords = [1, 2, 3, 4]; // 您新给出的IV值

const key = wordArrayToBuffer(keyWords);
const iv = wordArrayToBuffer(ivWords);

// 明文消息
const user_plain = '';
const password_plain = '';

// 创建并使用cipher实例进行加密
const cipher = crypto.createCipheriv('aes-128-cbc', key, iv);
let username = cipher.update(user_plain, 'utf8', 'hex');
username += cipher.final('hex');

const cipherp = crypto.createCipheriv('aes-128-cbc', key, iv);
let password = cipherp.update(password_plain, 'utf8', 'hex');
password += cipherp.final('hex')

console.log('用户名加密后的密文:', username);
console.log('用户名加密后的密文:', password);
