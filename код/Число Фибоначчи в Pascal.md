#Pascal #code 
```pascal
program numFib;
var
  x, y, q, z, i: integer;

begin
  x := 0;
  y := 1;
  write('Введите номер из числа последовательности Фибоначи: ');
  readln(z);
  for i := 3 to z do
    begin
      q := y;
      y := x + y;
      x := q;
    end;
    writeln('Число под номером ', z, ' равно ',y);
end.
```
