# Parâmetros do método `run()`

Documentação sobre como passar parâmetros para o método `run()` do `BubblewrapSandboxRunner` ou da facade `BubblewrapSandbox`.

## Assinatura

```php
run(array $command, array $extraBinds = [], $workingDirectory = null, array $env = null, $timeout = 60)
```

## Parâmetros

### 1. `$command` (obrigatório)
**Tipo:** `array`  
**Descrição:** Comando e seus argumentos como array. O primeiro elemento é o binário/executável, os demais são argumentos.

**Exemplos:**
```php
['echo', 'hello']
['ls', '-la', '/tmp']
['gs', '-sDEVICE=pdfwrite', '-dNOPAUSE', '-sOutputFile=output.pdf', 'input.pdf']
```

### 2. `$extraBinds` (opcional)
**Tipo:** `array`  
**Padrão:** `[]`  
**Descrição:** Montagens adicionais de diretórios/arquivos no sandbox. Cada item pode ser:
- **String simples:** caminho absoluto montado como somente leitura no mesmo caminho
- **Array associativo:** `['from' => '/caminho/host', 'to' => '/caminho/sandbox', 'read_only' => true]`

**Exemplos:**
```php
// String simples (somente leitura)
['/var/www/storage/input']

// Array com controle completo
[
    ['from' => '/var/www/storage/input', 'to' => '/var/www/storage/input', 'read_only' => true],
    ['from' => '/var/www/storage/output', 'to' => '/var/www/storage/output', 'read_only' => false]
]
```

**Propriedades do array associativo:**
- `from` (string, obrigatório): Caminho no host/sistema
- `to` (string, obrigatório): Caminho visto dentro do sandbox
- `read_only` (bool, opcional, padrão: `true`): `true` para somente leitura, `false` para permitir escrita

### 3. `$workingDirectory` (opcional)
**Tipo:** `string|null`  
**Padrão:** `null`  
**Descrição:** Diretório de trabalho dentro do sandbox. Deve ser um caminho absoluto. Se `null`, usa o diretório padrão do sandbox (`/tmp`).

**Exemplos:**
```php
'/var/www/storage/input'
'/tmp'
null  // usa o padrão
```

### 4. `$env` (opcional)
**Tipo:** `array|null`  
**Padrão:** `null`  
**Descrição:** Variáveis de ambiente adicionais para o processo executado no sandbox. Array associativo onde a chave é o nome da variável e o valor é o valor.

**Exemplos:**
```php
['LANG' => 'pt_BR.UTF-8', 'HOME' => '/tmp']
null  // sem variáveis extras
```

### 5. `$timeout` (opcional)
**Tipo:** `int|null`  
**Padrão:** `60`  
**Descrição:** Timeout em segundos. Após esse tempo, o processo é finalizado. Use `null` para não ter timeout.

**Exemplos:**
```php
30   // 30 segundos
120  // 2 minutos
null // sem timeout
```

## Exemplos de uso

### Exemplo básico
```php
$process = $sandbox->run(['echo', 'hello']);
echo $process->getOutput(); // "hello"
```

### Com binds e diretório de trabalho
```php
$binds = [
    ['from' => '/var/www/storage/input', 'to' => '/var/www/storage/input', 'read_only' => true],
    ['from' => '/var/www/storage/output', 'to' => '/var/www/storage/output', 'read_only' => false],
];

$process = $sandbox->run(
    ['heif-convert', '/var/www/storage/input/photo.heic', '/var/www/storage/output/photo.png'],
    $binds,
    '/var/www/storage/input',  // working directory
    null,                       // env
    60                          // timeout
);
```

### Com variáveis de ambiente
```php
$process = $sandbox->run(
    ['python3', 'script.py'],
    [],
    null,
    ['PYTHONPATH' => '/tmp', 'HOME' => '/tmp'],
    120
);
```

### Com timeout personalizado
```php
$process = $sandbox->run(
    ['gs', '-sDEVICE=pdfwrite', '-sOutputFile=output.pdf', 'input.pdf'],
    $binds,
    null,
    null,
    300  // 5 minutos
);
```

## Notas importantes

1. **Caminhos devem ser absolutos:** Todos os caminhos (`from`, `to`, `workingDirectory`) devem começar com `/`
2. **O comando é obrigatório:** O array `$command` não pode estar vazio
3. **Timeout:** Valores negativos não são permitidos
4. **Exceções:** O método lança exceção se o comando falhar (usa `mustRun()` internamente)

