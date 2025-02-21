# tec-MINT-Cross-Compiler
convert source to MINT


# C to MINT

matlabe code

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


 


