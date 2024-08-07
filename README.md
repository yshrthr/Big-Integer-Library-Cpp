#include <iostream>
#include <vector>
#include <string>
#include <algorithm>
#include <stdexcept>

class BigInt {
private:
    std::vector<int> digits;
    bool is_negative;

    void removeLeadingZeros() {
        while (digits.size() > 1 && digits.back() == 0) {
            digits.pop_back();
        }
        if (digits.size() == 1 && digits[0] == 0) {
            is_negative = false; // 0 is not negative
        }
    }

    static BigInt addAbsolute(const BigInt &a, const BigInt &b) {
        BigInt result;
        result.is_negative = false;

        int carry = 0;
        size_t max_length = std::max(a.digits.size(), b.digits.size());
        for (size_t i = 0; i < max_length || carry; ++i) {
            int sum = carry;
            if (i < a.digits.size()) sum += a.digits[i];
            if (i < b.digits.size()) sum += b.digits[i];

            result.digits.push_back(sum % 10);
            carry = sum / 10;
        }

        return result;
    }

    static BigInt subtractAbsolute(const BigInt &a, const BigInt &b) {
        if (a < b) {
            throw std::invalid_argument("Subtraction would result in a negative value");
        }

        BigInt result;
        result.is_negative = false;

        int carry = 0;
        for (size_t i = 0; i < a.digits.size(); ++i) {
            int diff = a.digits[i] - carry - (i < b.digits.size() ? b.digits[i] : 0);
            if (diff < 0) {
                diff += 10;
                carry = 1;
            } else {
                carry = 0;
            }
            result.digits.push_back(diff);
        }

        result.removeLeadingZeros();
        return result;
    }

public:
    BigInt() : is_negative(false) {
        digits.push_back(0);
    }

    BigInt(const std::string &str) {
        is_negative = str[0] == '-';
        for (int i = str.size() - 1; i >= is_negative; --i) {
            if (!isdigit(str[i])) {
                throw std::invalid_argument("Invalid character in BigInt string");
            }
            digits.push_back(str[i] - '0');
        }
        removeLeadingZeros();
    }

    BigInt(long long num) {
        is_negative = num < 0;
        if (num == 0) {
            digits.push_back(0);
        } else {
            if (is_negative) num = -num;
            while (num > 0) {
                digits.push_back(num % 10);
                num /= 10;
            }
        }
    }

    friend std::ostream& operator<<(std::ostream &out, const BigInt &bigint) {
        if (bigint.is_negative) out << '-';
        for (int i = bigint.digits.size() - 1; i >= 0; --i) {
            out << bigint.digits[i];
        }
        return out;
    }

    bool operator<(const BigInt &other) const {
        if (is_negative != other.is_negative) return is_negative;
        if (digits.size() != other.digits.size()) {
            return digits.size() < other.digits.size() != is_negative;
        }
        for (int i = digits.size() - 1; i >= 0; --i) {
            if (digits[i] != other.digits[i]) {
                return digits[i] < other.digits[i] != is_negative;
            }
        }
        return false;
    }

    bool operator==(const BigInt &other) const {
        return is_negative == other.is_negative && digits == other.digits;
    }

    BigInt operator+(const BigInt &other) const {
        if (is_negative == other.is_negative) {
            BigInt result = addAbsolute(*this, other);
            result.is_negative = is_negative;
            return result;
        } else if (abs(*this) >= abs(other)) {
            BigInt result = subtractAbsolute(*this, other);
            result.is_negative = is_negative;
            return result;
        } else {
            BigInt result = subtractAbsolute(other, *this);
            result.is_negative = other.is_negative;
            return result;
        }
    }

    BigInt operator-(const BigInt &other) const {
        if (is_negative != other.is_negative) {
            BigInt result = addAbsolute(*this, other);
            result.is_negative = is_negative;
            return result;
        } else if (abs(*this) >= abs(other)) {
            BigInt result = subtractAbsolute(*this, other);
            result.is_negative = is_negative;
            return result;
        } else {
            BigInt result = subtractAbsolute(other, *this);
            result.is_negative = !other.is_negative;
            return result;
        }
    }

    BigInt operator*(const BigInt &other) const {
        BigInt result;
        result.digits.resize(digits.size() + other.digits.size(), 0);
        result.is_negative = is_negative != other.is_negative;

        for (size_t i = 0; i < digits.size(); ++i) {
            int carry = 0;
            for (size_t j = 0; j < other.digits.size() || carry; ++j) {
                long long mul = result.digits[i + j] + carry +
                    digits[i] * (j < other.digits.size() ? other.digits[j] : 0);
                result.digits[i + j] = mul % 10;
                carry = mul / 10;
            }
        }

        result.removeLeadingZeros();
        return result;
    }

    BigInt operator/(const BigInt &other) const {
        if (other == BigInt(0)) {
            throw std::invalid_argument("Division by zero");
        }

        BigInt dividend = abs(*this);
        BigInt divisor = abs(other);
        BigInt result;
        result.digits.resize(dividend.digits.size(), 0);

        BigInt current;
        for (int i = dividend.digits.size() - 1; i >= 0; --i) {
            current.digits.insert(current.digits.begin(), dividend.digits[i]);
            current.removeLeadingZeros();

            int count = 0;
            while (current >= divisor) {
                current = current - divisor;
                ++count;
            }
            result.digits[i] = count;
        }

        result.is_negative = is_negative != other.is_negative;
        result.removeLeadingZeros();
        return result;
    }

    static BigInt abs(const BigInt &num) {
        BigInt result = num;
        result.is_negative = false;
        return result;
    }

    friend BigInt operator%(const BigInt &a, const BigInt &b) {
        BigInt quotient = a / b;
        return a - quotient * b;
    }
};

int main() {
    BigInt a("-123456789123456789");
    BigInt b("987654321987654321");

    std::cout << "a: " << a << std::endl;
    std::cout << "b: " << b << std::endl;

    std::cout << "a + b: " << a + b << std::endl;
    std::cout << "a - b: " << a - b << std::endl;
    std::cout << "a * b: " << a * b << std::endl;
    std::cout << "b / a: " << b / a << std::endl;
    std::cout << "b % a: " << b % a << std::endl;

    return 0;
}
