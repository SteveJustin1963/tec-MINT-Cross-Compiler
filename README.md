# tec-MINT-Cross-Compiler
convert source to MINT


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

