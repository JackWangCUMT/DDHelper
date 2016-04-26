# DDHelper
阿里巴巴钉钉签名生成.NET帮助类
## 钉钉客户端开发JS-API认证 ##

**钉钉（DingTalk）** 是阿里巴巴集团专为中国企业打造的免费沟通和协同的多端平台，提供PC版，Web版和手机版，支持手机和电脑间文件互传 。
钉钉因中国企业而生，帮助中国企业通过系统化的解决方案（微应用），全方位提升中国企业沟通和协同效率。

###JS-API权限认证调试###
开发者在web页面使用钉钉容器提供的jsapi时，需要验证调用权限，并以参数signature标识合法性。我们可以将相关参数准备好，然后在官网 **https://debug.dingtalk.com** 上生成签名Signature.

###1签名生成的规则###
- 按照**参数字段名（不是参数值哈）**字典序排序 ： List keyArray = sort(noncestr, timestamp, jsapi_ticket, url);
- 拼接字符串（key1=value1&key2=value2...）： String str = assemble(keyArray);
- 计算sh1值：signature = sha1(str);

###.NET 签名生成算法###
```

    public static class DingTalkAuth
    {
        /// <summary>
        ///开发者在web页面使用钉钉容器提供的jsapi时，需要验证调用权限，并以参数signature标识合法性
        ///签名生成的规则：
        ///List keyArray = sort(noncestr, timestamp, jsapi_ticket, url);
        /// String str = assemble(keyArray);
        ///signature = sha1(str);
        /// </summary>
        /// <param name="noncestr">随机字符串，自己随便填写即可</param>
        /// <param name="sTimeStamp">当前时间戳，具体值为当前时间到1970年1月1号的秒数</param>
        /// <param name="jsapi_ticket">获取的jsapi_ticket</param>
        /// <param name="url">当前网页的URL，不包含#及其后面部分</param>
        /// <param name="signature">生成的签名</param>
        /// <returns>0 成功，2 失败</returns>
        public static int GenSigurate(string noncestr, string sTimeStamp, string jsapi_ticket, string url, ref string signature)
        {


            //例如：
            //noncestr = Zn4zmLFKD0wzilzM
            //jsapi_ticket = mS5k98fdkdgDKxkXGEs8LORVREiweeWETE40P37wkidkfksDSKDJFD5h9nbSlYy3-Sl-HhTdfl2fzFy1AOcKIDU8l
            //timestamp = 1414588745
            //url = http://open.dingtalk.com

            //步骤1.sort()含义为对所有待签名参数按照字段名的ASCII 码从小到大排序（字典序）
            //注意，此处是是按照【字段名】的ASCII字典序，而不是参数值的字典序（这个细节折磨我很久了)
            //0:jsapi_ticket 1:noncestr 2:timestamp 3:url;

            //步骤2.assemble()含义为根据步骤1中获的参数字段的顺序，使用URL键值对的格式（即key1 = value1 & key2 = value2…）拼接成字符串
            //string assemble = "jsapi_ticket=3fOo5UfWhmvRKnRGMmm6cWwmIxDMCnniyVYL2fqcz1I4GNU4054IOlif0dZjDaXUScEjoOnJWOVrdwTCkYrwSl&noncestr=CUMT1987wlrrlw&timestamp=1461565921&url=https://jackwangcumt.github.io/home.html";
            string assemble =string.Format("jsapi_ticket={0}&noncestr={1}&timestamp={2}&url={3}", jsapi_ticket, noncestr,sTimeStamp, url);
            //步骤2.sha1()的含义为对在步骤2拼接好的字符串进行sha1加密。
            SHA1 sha;
            ASCIIEncoding enc;
            string hash = "";
            try
            {
                sha = new SHA1CryptoServiceProvider();
                enc = new ASCIIEncoding();
                byte[] dataToHash = enc.GetBytes(assemble);
                byte[] dataHashed = sha.ComputeHash(dataToHash);
                hash = BitConverter.ToString(dataHashed).Replace("-", "");
                hash = hash.ToLower();
            }
            catch (Exception)
            {
                return 2;
            }
            signature = hash;
            return 0;
           
        }

        /// <summary>
        /// 获取时间戳timestamp（当前时间戳，具体值为当前时间到1970年1月1号的秒数）
        /// </summary>
        /// <returns></returns>
        public static string GetTimeStamp()
        {
            TimeSpan ts = DateTime.UtcNow - new DateTime(1970, 1, 1, 0, 0, 0, 0);
            return Convert.ToInt64(ts.TotalSeconds).ToString();
        }
        /// <summary>
        /// 字典排序
        /// </summary>
        public class DictionarySort : System.Collections.IComparer
        {
            public int Compare(object oLeft, object oRight)
            {
                string sLeft = oLeft as string;
                string sRight = oRight as string;
                int iLeftLength = sLeft.Length;
                int iRightLength = sRight.Length;
                int index = 0;
                while (index < iLeftLength && index < iRightLength)
                {
                    if (sLeft[index] < sRight[index])
                        return -1;
                    else if (sLeft[index] > sRight[index])
                        return 1;
                    else
                        index++;
                }
                return iLeftLength - iRightLength;

            }
        }
    }
}
```

