# tec-MINT-Cross-Compiler
- C to MINT
- MINT to Excel

# C to MINT

## Matlab 

```
function test()
    % Test function for C to MINT converter
    
    % Sample C code - using actual newlines
    cCode = ['int main() {' newline ...
             '    int x = 5;' newline ...
             '    int y = 10;' newline ...
             '    int z;' newline ...
             '    z = x + y;' newline ...
             '    printf("Sum is: ");' newline ...
             '    printf("%d", z);' newline ...
             '}'];
    
    % Convert C code to MINT
    mintCode = c2mint(cCode);
    
    % Display results
    disp('Original C code:');
    disp(cCode);
    disp('Converted MINT code:');
    disp(mintCode);
end

function mintCode = c2mint(cCode)
    % Split code into lines and initialize output
    lines = splitlines(cCode);
    mintCode = ':A ';  % Using A as function name instead of 'main'
    
    % Process each line
    for i = 1:length(lines)
        line = strtrim(lines{i});
        
        % Skip empty lines, function declaration, and closing brace
        if isempty(line) || contains(line, 'main()') || strcmp(line, '}')
            continue;
        end
        
        % Remove semicolons
        line = strrep(line, ';', '');
        
        % Process line based on content
        if contains(line, 'int ')
            % Variable declaration and initialization
            line = strrep(line, 'int ', '');
            if contains(line, '=')
                % Has initialization
                parts = strsplit(line, '=');
                var = strtrim(parts{1});
                val = strtrim(parts{2});
                mintCode = [mintCode val ' ' var ' ! '];
            else
                % Just declaration
                var = strtrim(line);
                mintCode = [mintCode '0 ' var ' ! '];
            end
        elseif contains(line, '=')
            % Assignment
            parts = strsplit(line, '=');
            var = strtrim(parts{1});
            expr = strtrim(parts{2});
            
            % Handle addition
            if contains(expr, '+')
                ops = strsplit(expr, '+');
                mintCode = [mintCode strtrim(ops{1}) ' ' strtrim(ops{2}) ' + ' var ' ! '];
            else
                mintCode = [mintCode expr ' ' var ' ! '];
            end
        elseif contains(line, 'printf')
            % Print statement
            if contains(line, '%d')
                % Printing variable
                var = extractBetween(line, '%d", ', ')');
                if ~isempty(var)
                    mintCode = [mintCode var{1} ' . '];
                end
            else
                % Printing string
                str = extractBetween(line, '"', '"');
                if ~isempty(str)
                    mintCode = [mintCode '`' str{1} '` '];
                end
            end
        end
    end
    
    % Add function terminator
    mintCode = [mintCode ';'];
    
    % Clean up spacing
    mintCode = regexprep(mintCode, '\s+', ' ');
    mintCode = strtrim(mintCode);
end
```

```
>> test in matlab
Original C code:
int main() {
    int x = 5;
    int y = 10;
    int z;
    z = x + y;
    printf("Sum is: ");
    printf("%d", z);
}
Converted MINT code:
:A 5 x ! 10 y ! 0 z ! x y + z ! `Sum is: ` z . ;
>
```

lets run it in MINT

```
> A
Sum is: 15
>
```

it works!!!


 
## Python -  C to Mint

 
 ```
class C2MintConverter:
    def __init__(self):
        self.var_counter = 0
        self.func_counter = 0
        self.available_vars = list('abcdefghijklmnopqrstuvwxyz')
        self.available_funcs = list('ABCDEFGHIJKLMNOPQRSTUVWXY')  # Z reserved for interrupts
        self.current_vars = {}
        
    def convert(self, c_code: str) -> str:
        """Convert C code to MINT code."""
        # Split into lines and clean
        lines = [line.strip() for line in c_code.split('\n')]
        lines = [line for line in lines if line and not line.isspace()]
        
        mint_code = []
        # Start with function A
        mint_code.append(':A')
        
        in_function = False
        for line in lines:
            # Skip empty lines and comments
            if not line or line.startswith('//'):
                continue
                
            # Remove semicolons and curly braces
            line = line.replace(';', '').replace('{', '').replace('}', '').strip()
            
            # Skip main function declaration
            if 'main()' in line:
                continue
                
            # Process line based on type
            if 'int ' in line:
                mint_code.extend(self._handle_declaration(line))
            elif '=' in line:
                mint_code.extend(self._handle_assignment(line))
            elif 'printf' in line:
                mint_code.extend(self._handle_printf(line))
            elif 'for' in line:
                mint_code.extend(self._handle_for_loop(line))
            elif 'while' in line:
                mint_code.extend(self._handle_while_loop(line))
            elif 'if' in line:
                mint_code.extend(self._handle_if_statement(line))
                
        # Add function terminator
        mint_code.append(';')
        
        # Join and clean up
        return ' '.join(mint_code)
    
    def _handle_declaration(self, line: str) -> list:
        """Handle variable declarations."""
        parts = line.replace('int ', '').split('=')
        var_name = parts[0].strip()
        
        # Assign a MINT variable name
        mint_var = self._get_var_name(var_name)
        
        if len(parts) > 1:
            # Has initialization
            value = parts[1].strip()
            return [value, mint_var, '!']
        else:
            # Just declaration, initialize to 0
            return ['0', mint_var, '!']
    
    def _handle_assignment(self, line: str) -> list:
        """Handle variable assignments."""
        parts = line.split('=')
        var_name = parts[0].strip()
        expr = parts[1].strip()
        
        mint_var = self.current_vars.get(var_name, self._get_var_name(var_name))
        
        # Handle different types of expressions
        if '+' in expr:
            ops = expr.split('+')
            ops = [self.current_vars.get(op.strip(), op.strip()) for op in ops]
            return [ops[0], ops[1], '+', mint_var, '!']
        elif '-' in expr:
            ops = expr.split('-')
            ops = [self.current_vars.get(op.strip(), op.strip()) for op in ops]
            return [ops[0], ops[1], '-', mint_var, '!']
        elif '*' in expr:
            ops = expr.split('*')
            ops = [self.current_vars.get(op.strip(), op.strip()) for op in ops]
            return [ops[0], ops[1], '*', mint_var, '!']
        elif '/' in expr:
            ops = expr.split('/')
            ops = [self.current_vars.get(op.strip(), op.strip()) for op in ops]
            return [ops[0], ops[1], '/', mint_var, '!']
        else:
            # Simple assignment
            value = self.current_vars.get(expr, expr)
            return [value, mint_var, '!']
    
    def _handle_printf(self, line: str) -> list:
        """Handle printf statements."""
        if '%d' in line:
            # Printing a variable
            var = line[line.find('%d", ')+4:line.find(')')].strip()
            mint_var = self.current_vars.get(var, var)
            return [mint_var, '.']
        else:
            # Printing a string
            start = line.find('"') + 1
            end = line.rfind('"')
            text = line[start:end]
            return [f'`{text}`']
    
    def _handle_for_loop(self, line: str) -> list:
        """Handle for loops."""
        # Extract loop parameters
        start = line.find('(') + 1
        end = line.rfind(')')
        params = line[start:end].split(';')
        
        init = params[0].strip()
        condition = params[1].strip()
        increment = params[2].strip()
        
        # Convert to MINT format
        mint_code = []
        # Handle initialization
        if '=' in init:
            mint_code.extend(self._handle_assignment(init))
            
        # Extract loop count from condition
        count = condition.split('<')[1].strip()
        mint_code.append(count)
        mint_code.append('(')
        
        return mint_code
    
    def _handle_while_loop(self, line: str) -> list:
        """Handle while loops."""
        # Extract condition
        start = line.find('(') + 1
        end = line.rfind(')')
        condition = line[start:end].strip()
        
        return ['/U', '(', condition, '/W']
    
    def _handle_if_statement(self, line: str) -> list:
        """Handle if statements."""
        # Extract condition
        start = line.find('(') + 1
        end = line.rfind(')')
        condition = line[start:end].strip()
        
        return [condition, '(']
    
    def _get_var_name(self, c_var: str) -> str:
        """Get or create a MINT variable name."""
        if c_var not in self.current_vars:
            if not self.available_vars:
                raise ValueError("No more variables available")
            self.current_vars[c_var] = self.available_vars.pop(0)
        return self.current_vars[c_var]

def test_converter():
    """Test function with sample C code."""
    # Sample C code
    c_code = """
    int main() {
        int x = 5;
        int y = 10;
        int z;
        z = x + y;
        printf("Sum is: ");
        printf("%d", z);
    }
    """
    
    # Create converter and convert code
    converter = C2MintConverter()
    mint_code = converter.convert(c_code)
    
    # Print results
    print("Original C code:")
    print(c_code)
    print("\nConverted MINT code:")
    print(mint_code)

if __name__ == "__main__":
    test_converter()
```

```
Original C code:

    int main() {
        int x = 5;
        int y = 10;
        int z;
        z = x + y;
        printf("Sum is: ");
        printf("%d", z);
    }
    

Converted MINT code:
:A 5 a ! 10 b ! 0 c ! a b + c ! `Sum is: ` c . ;

=== Code Execution Successful ===
```

# Comprehensive Python program

Create a comprehensive Python program that converts C code to MINT, including standard I/O and math operations.


This compiler handles:

1. Basic C syntax:
   - Variable declarations and assignments
   - Arithmetic operations
   - Control structures (if, while, for)
   - Function definitions
   - Return statements

2. Standard I/O:
   - printf
   - scanf
   - Basic string handling

3. Math operations:
   - Basic arithmetic (+, -, *, /)
   - Bitwise operations (&, |, ^, ~, <<, >>)
   - Basic math functions (abs, pow)
   - Remainders using MINT's /r variable

4. MINT-specific features:
   - Single-letter variables (a-z)
   - Single-letter functions (A-Y)
   - RPN notation
   - Stack-based operations
   - Array handling

To use it:
```python
# Create compiler instance
compiler = MintCompiler()

# Example C code
c_code = """
#include <stdio.h>
int main() {
    int x = 5;
    int y = 10;
    int z = x + y;
    printf("Sum is: %d\\n", z);
    return 0;
}
"""

# Compile to MINT
mint_code = compiler.compile(c_code)
print(mint_code)
```







```
import re
from typing import List, Dict

class MintCompiler:
    def __init__(self):
        # Available MINT variables and functions
        self.vars = list('abcdefghijklmnopqrstuvwxyz')
        self.funcs = list('ABCDEFGHIJKLMNOPQRSTUVWXY')  # Z reserved for interrupts
        self.used_vars = {}
        self.used_funcs = {}
        self.current_function = None
        
        # C to MINT operation mappings
        self.math_ops = {
            '+': '+',
            '-': '-',
            '*': '*',
            '/': '/',
            '%': '/',  # Division remainder stored in /r
            '<<': '{',
            '>>': '}',
            '&': '&',
            '|': '|',
            '^': '^',
            '~': '~'
        }
        
        # C standard library function mappings
        self.stdlib_funcs = {
            'printf': self._handle_printf,
            'scanf': self._handle_scanf,
            'strlen': self._handle_strlen,
            'abs': self._handle_abs,
            'pow': self._handle_pow
        }

    def compile(self, c_code: str) -> str:
        """Main compilation function"""
        # Preprocess code
        c_code = self._preprocess(c_code)
        
        # Split into functions
        functions = self._split_functions(c_code)
        
        # Compile each function
        mint_code = []
        for func_name, func_body in functions.items():
            mint_func = self._compile_function(func_name, func_body)
            mint_code.append(mint_func)
        
        return '\n'.join(mint_code)

    def _preprocess(self, code: str) -> str:
        """Preprocess C code"""
        # Remove comments
        code = re.sub(r'//.*?\n|/\*.*?\*/', '', code, flags=re.S)
        # Remove includes
        code = re.sub(r'#include.*?\n', '', code)
        # Remove extra whitespace
        code = ' '.join(code.split())
        return code

    def _split_functions(self, code: str) -> Dict[str, str]:
        """Split C code into separate functions"""
        functions = {}
        # Find function definitions
        func_pattern = r'(\w+)\s+(\w+)\s*\((.*?)\)\s*{(.*?)}'
        matches = re.finditer(func_pattern, code, re.DOTALL)
        
        for match in matches:
            ret_type, name, params, body = match.groups()
            functions[name] = {
                'return_type': ret_type,
                'params': params,
                'body': body
            }
        
        return functions

    def _compile_function(self, name: str, func_info: dict) -> str:
        """Compile a single function to MINT"""
        if name not in self.used_funcs:
            if not self.funcs:
                raise ValueError("No more function names available")
            self.used_funcs[name] = self.funcs.pop(0)
        
        mint_name = self.used_funcs[name]
        self.current_function = mint_name
        
        # Convert body
        body = self._compile_block(func_info['body'])
        
        return f':{mint_name} {body} ;'

    def _compile_block(self, block: str) -> str:
        """Compile a block of C code to MINT"""
        lines = [line.strip() for line in block.split(';') if line.strip()]
        mint_lines = []
        
        for line in lines:
            if '=' in line and not line.startswith('if') and not line.startswith('for'):
                mint_lines.extend(self._handle_assignment(line))
            elif any(op in line for op in self.math_ops):
                mint_lines.extend(self._handle_expression(line))
            elif any(func in line for func in self.stdlib_funcs):
                mint_lines.extend(self._handle_stdlib_call(line))
            elif line.startswith('for'):
                mint_lines.extend(self._handle_for_loop(line))
            elif line.startswith('while'):
                mint_lines.extend(self._handle_while_loop(line))
            elif line.startswith('if'):
                mint_lines.extend(self._handle_if_statement(line))
            elif line.startswith('return'):
                continue  # Skip return statements for now
                
        return ' '.join(mint_lines)

    def _handle_assignment(self, line: str) -> List[str]:
        """Handle variable assignment"""
        if 'int' in line:
            line = line.replace('int', '').strip()
            
        parts = line.split('=')
        var = parts[0].strip()
        
        # Handle array declarations
        if '[' in var:
            return self._handle_array_declaration(line)
            
        # Get or create MINT variable name
        if var not in self.used_vars:
            if not self.vars:
                raise ValueError("No more variable names available")
            self.used_vars[var] = self.vars.pop(0)
        
        mint_var = self.used_vars[var]
        
        if len(parts) == 1:  # Just declaration
            return ['0', mint_var, '!']
            
        expr = parts[1].strip()
        expr_code = self._handle_expression(expr)
        
        return expr_code + [mint_var, '!']

    def _handle_array_declaration(self, line: str) -> List[str]:
        """Handle array declaration and initialization"""
        match = re.match(r'int\s+(\w+)\s*\[\s*\]\s*=\s*{(.*)}', line)
        if match:
            var_name, values = match.groups()
            values = [v.strip() for v in values.split(',')]
            mint_var = self._get_var_name(var_name)
            return ['['] + values + [']', mint_var, '!']
        return []

    def _handle_expression(self, expr: str) -> List[str]:
        """Convert C expression to MINT RPN"""
        tokens = self._tokenize(expr)
        return self._to_rpn(tokens)

    def _handle_stdlib_call(self, line: str) -> List[str]:
        """Handle C standard library function calls"""
        func_name = re.match(r'(\w+)\(', line).group(1)
        if func_name in self.stdlib_funcs:
            return self.stdlib_funcs[func_name](line)
        return []

    def _handle_printf(self, line: str) -> List[str]:
        """Handle printf function"""
        match = re.match(r'printf\("([^"]*)"(.*)\)', line)
        if not match:
            return []
            
        fmt, args = match.groups()
        mint_code = []
        
        if fmt:
            mint_code.append(f'`{fmt}`')
            
        if args:
            args = [arg.strip() for arg in args.split(',')[1:]]
            for arg in args:
                if arg in self.used_vars:
                    mint_code.extend([self.used_vars[arg], '.'])
                else:
                    mint_code.extend([arg, '.'])
                    
        return mint_code

    def _handle_scanf(self, line: str) -> List[str]:
        """Handle scanf function"""
        return ['/K']

    def _handle_strlen(self, line: str) -> List[str]:
        """Handle strlen function"""
        var = re.search(r'strlen\((.*?)\)', line).group(1)
        return [self.used_vars.get(var, var), '/S']

    def _handle_abs(self, line: str) -> List[str]:
        """Handle abs function"""
        var = re.search(r'abs\((.*?)\)', line).group(1)
        mint_var = self.used_vars.get(var, var)
        return [mint_var, '"', '0', '<', '(', '-1', '*', ')', '/E', mint_var]

    def _handle_pow(self, line: str) -> List[str]:
        """Handle power function"""
        match = re.search(r'pow\((.*?),(.*?)\)', line)
        base, exp = match.groups()
        return ['1', 't', '!', exp, '(', 't', base, '*', 't', '!', ')']

    def _handle_for_loop(self, line: str) -> List[str]:
        """Handle for loop"""
        # Extract loop parameters
        params = re.search(r'for\s*\((.*?);(.*?);(.*?)\)', line).groups()
        init, cond, incr = [p.strip() for p in params]
        
        mint_code = []
        # Handle initialization
        if '=' in init:
            mint_code.extend(self._handle_assignment(init))
            
        # Handle condition and create loop
        if '<' in cond:
            limit = cond.split('<')[1].strip()
            mint_code.extend([limit, '('])
            
        return mint_code

    def _handle_while_loop(self, line: str) -> List[str]:
        """Handle while loop"""
        cond = re.search(r'while\s*\((.*?)\)', line).group(1)
        return ['/U', '(', cond, '/W']

    def _handle_if_statement(self, line: str) -> List[str]:
        """Handle if statement"""
        cond = re.search(r'if\s*\((.*?)\)', line).group(1)
        return [cond, '(']

    def _tokenize(self, expr: str) -> List[str]:
        """Tokenize C expression"""
        tokens = []
        current = ''
        
        for char in expr:
            if char.isspace():
                if current:
                    tokens.append(current)
                    current = ''
            elif char in self.math_ops:
                if current:
                    tokens.append(current)
                    current = ''
                tokens.append(char)
            else:
                current += char
                
        if current:
            tokens.append(current)
            
        return tokens

    def _to_rpn(self, tokens: List[str]) -> List[str]:
        """Convert tokens to RPN"""
        precedence = {
            '+': 1, '-': 1,
            '*': 2, '/': 2,
            '<<': 3, '>>': 3,
            '&': 4, '|': 4, '^': 4
        }
        
        output = []
        operators = []
        
        for token in tokens:
            if token in self.math_ops:
                while (operators and operators[-1] in precedence and 
                       precedence[operators[-1]] >= precedence[token]):
                    output.append(self.math_ops[operators.pop()])
                operators.append(token)
            else:
                if token in self.used_vars:
                    output.append(self.used_vars[token])
                else:
                    output.append(token)
        
        while operators:
            output.append(self.math_ops[operators.pop()])
            
        return output

    def _get_var_name(self, c_var: str) -> str:
        """Get or create a MINT variable name"""
        if c_var not in self.used_vars:
            if not self.vars:
                raise ValueError("No more variables available")
            self.used_vars[c_var] = self.vars.pop(0)
        return self.used_vars[c_var]

def test_compiler():
    # Test with simple C code
    c_code = """
    #include <stdio.h>
    int main() {
        int x = 5;
        int y = 10;
        int z = x + y;
        printf("Sum is: %d\\n", z);
        return 0;
    }
    """
    
    compiler = MintCompiler()
    mint_code = compiler.compile(c_code)
    print("C code:")
    print(c_code)
    print("\nMINT code:")
    print(mint_code)

if __name__ == "__main__":
    test_compiler()
```

### test

```
C code:

    #include <stdio.h>
    int main() {
        int x = 5;
        int y = 10;
        int z = x + y;
        printf("Sum is: %d\n", z);
        return 0;
    }
    

MINT code:
:A 5 a ! 10 b ! a b + c ! printf("Sum is: d\n", z) / ;

=== Code Execution Successful ===
```


# MINT in EXCEL

# MINT Code Processor in Excel

## Spreadsheet Layout

### Column Structure
```
A: Line Number (1,2,3...)
B: MINT Code (one command per line)
C: Stack State 
D: Output
E: Carry Flag (/c)
F: Remainder/Roll Flag (/r)
G-AF: Variables (a through z)
```

### Cell Formulas

#### Line Numbers (Column A)
```excel
A2: =1
A3: =IF(B3<>"", A2+1, "")  [Fill down]
```

#### Stack State (Column C)
```excel
C2: =IF(B2="", "",
    SWITCH(RIGHT(B2,1),
        "+", ProcessAdd(GetStack(C1)),
        "-", ProcessSubtract(GetStack(C1)),
        "*", ProcessMultiply(GetStack(C1)),
        "/", ProcessDivide(GetStack(C1)),
        """", DuplicateTop(GetStack(C1)),
        "$", SwapTop(GetStack(C1)),
        "!", StoreVariable(GetStack(C1), LEFT(B2,1)),
        IF(ISNUMBER(VALUE(B2)), PushStack(GetStack(C1), B2), GetStack(C1))
    ))
```

#### Output Display (Column D)
```excel
D2: =IF(OR(RIGHT(B2,1)=".", RIGHT(B2,1)=","),
    IF(RIGHT(B2,1)=".",
        VALUE(LEFT(GetStack(C2), FIND(",", GetStack(C2)&",") - 1)),
        DEC2HEX(VALUE(LEFT(GetStack(C2), FIND(",", GetStack(C2)&",") - 1)))
    ),
    "")
```

#### Carry Flag (Column E)
```excel
E2: =IF(B2="",E1,
    IF(OR(RIGHT(B2,1)="+",RIGHT(B2,1)="-"),
        IF(ABS(VALUE(ProcessOperation(GetStack(C1),RIGHT(B2,1))))>32767,1,0),
        IF(B2="0 /c!",0,E1)))
```

#### Remainder Flag (Column F)
```excel
F2: =IF(B2="",F1,
    IF(RIGHT(B2,1)="/",
        MOD(VALUE(LEFT(GetStack(C1), FIND(",", GetStack(C1)&",") - 1)),
            VALUE(MID(GetStack(C1), FIND(",", GetStack(C1)) + 1, 999))),
        IF(B2="0 /r!",0,F1)))
```

#### Variables (Columns G-AF)
```excel
G2: =IF(AND(RIGHT(B2,1)="!", LEFT(B2,1)="a"),
        VALUE(LEFT(GetStack(C1), FIND(",", GetStack(C1)&",") - 1)),
        G1)
```
[Similar formulas for other variables b-z]

### Helper Functions (VBA)

```vba
Function GetStack(prevStack As String) As String
    ' Returns current stack state
End Function

Function PushStack(stack As String, value As String) As String
    ' Pushes a value onto the stack
End Function

Function ProcessAdd(stack As String) As String
    ' Processes addition operation
End Function

' [Additional helper functions for other operations]
```

## Usage Example

Here's how the spreadsheet would process a simple MINT program:

```
B2: 10        C2: 10         D2:          E2: 0    F2: 0
B3: 20        C3: 10,20      D3:          E3: 0    F3: 0
B4: +         C4: 30         D4:          E4: 0    F4: 0
B5: .         C5: 30         D5: 30       E5: 0    F5: 0
```

## Advanced Features

### Error Handling
```excel
' Add to each operation formula
=IF(CountStack(C1)<RequiredOperands(B2), "Error: Stack underflow", ...)
```

### Stack Validation
```excel
' Add to push operations
=IF(LEN(C1)>255, "Error: Stack overflow", ...)
```

### Variable Type Checking
```excel
' Add to variable operations
=IF(NOT(ISNUMBER(VALUE(GetStack(C1)))), "Error: Type mismatch", ...)
```

## Implementation Notes

1. Each row processes one MINT command
2. State is maintained through cell references to previous row
3. Stack is represented as comma-separated values
4. Variables persist until explicitly changed
5. Flags update based on operations

## Limitations

1. Maximum stack depth limited by Excel cell character limit
2. No support for nested arrays
3. Limited function definition capability
4. No direct loop support
5. 16-bit integer operations only

## Future Enhancements

1. Support for arrays
2. Function definitions
3. Loop handling
4. Debugging capabilities
5. Step-through execution

   
