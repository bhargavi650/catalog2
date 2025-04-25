# catalog2
import java.io.FileReader;
import java.io.IOException;
import java.math.BigInteger;
import org.json.simple.JSONObject;
import org.json.simple.parser.JSONParser;
import org.json.simple.parser.ParseException;

public class ShamirSecretSharing {

    public static void main(String[] args) throws IOException, ParseException {
        JSONParser parser = new JSONParser();
        JSONObject testCase1 = (JSONObject) parser.parse(new FileReader("testcase1.json"));
        JSONObject testCase2 = (JSONObject) parser.parse(new FileReader("testcase2.json"));

        BigInteger secret1 = findSecret(testCase1);
        BigInteger secret2 = findSecret(testCase2);

        System.out.println("Secret for Test Case 1: " + secret1);
        System.out.println("Secret for Test Case 2: " + secret2);
    }

    public static BigInteger findSecret(JSONObject json) {
        JSONObject keys = (JSONObject) json.get("keys");
        long k = (long) keys.get("k");
        BigInteger[] x = new BigInteger[(int) k];
        BigInteger[] y = new BigInteger[(int) k];

        int i = 0;
        for (Object key : json.keySet()) {
            if (!key.equals("keys") && i < k) {
                JSONObject point = (JSONObject) json.get(key);
                x[i] = BigInteger.valueOf(Long.parseLong(key.toString()));
                y[i] = new BigInteger(point.get("value").toString(), Integer.parseInt(point.get("base").toString()));
                i++;
            }
        }

        return lagrangeInterpolation(x, y, BigInteger.ZERO);
    }

    public static BigInteger lagrangeInterpolation(BigInteger[] x, BigInteger[] y, BigInteger xInterp) {
        BigInteger result = BigInteger.ZERO;
        for (int i = 0; i < x.length; i++) {
            BigInteger numerator = BigInteger.ONE;
            BigInteger denominator = BigInteger.ONE;
            for (int j = 0; j < x.length; j++) {
                if (i != j) {
                    numerator = numerator.multiply(xInterp.subtract(x[j]));
                    denominator = denominator.multiply(x[i].subtract(x[j]));
                }
            }
            BigInteger term = y[i].multiply(numerator).divide(denominator);
            result = result.add(term);
        }
        return result;
    }
}

