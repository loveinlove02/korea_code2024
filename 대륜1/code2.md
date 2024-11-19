```python
# 한글 문자열
hangul_string = "도움이 필요합니다."

# 유니코드로 변환
unicode_string = ""
for char in hangul_string:
    unicode_string +=format(ord(char), "04x")

print("한글 문자열:", hangul_string)
print("유니코드:", unicode_string.upper())
```
