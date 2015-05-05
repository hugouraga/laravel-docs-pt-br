# Contribution Guide

- [Relatar Bugs](#bug-reports)
- [Discurssão sobre o Desenvolvimento do Core](#core-development-discussion)
- [Quial Branch?](#which-branch)
- [Vulnerabilidades de Segurança](#security-vulnerabilities)
- [Estilo de Código](#coding-style)

<a name="bug-reports"></a>
## Relatar Reports

Para encoragar a ativa colaboração, Laravel encoraja fortemente pull requests, não apenas relatório de bugs. "Relatar Bugs" pode também ser enviada em forma de pull request contendo um teste unitário que falhou. 

No entanto, se você enviar um relatório de bug, o problema deve conter um título e uma descrição clara do problema. Você deve incluir o máximo de informação relevante possível e uma amostra do código que mostra o problema. O objetivo de relatório de bugs é tonar mais fácil para si mesmo - e outros - replicar o bug e desenvolver a solução. 

Lembre-se, relatório de bugs foi criada com o intúito que outras pessoas com o seu mesmo problemas irão ser aptas de colaborar com você para resolver isto. Não espere que a relatório de bug verá qualquer atividade ou qualquer outra coisa e irá pular par resolver isto. Criar um relatório bug serve para ajudar a você mesmo e outros que começaram no caminho da solução de problemas. 

O código fonte do Laravel é gerenciado no Github, e lá existem repositórios para cada projeto Laravel:

- [Laravel Framework](https://github.com/laravel/framework)
- [Laravel Application](https://github.com/laravel/laravel)
- [Laravel Documentation](https://github.com/laravel/docs)
- [Laravel Cashier](https://github.com/laravel/cashier)
- [Laravel Envoy](https://github.com/laravel/envoy)
- [Laravel Homestead](https://github.com/laravel/homestead)
- [Laravel Homestead Build Scripts](https://github.com/laravel/settler)
- [Laravel Website](https://github.com/laravel/laravel.com)
- [Laravel Art](https://github.com/laravel/art)

<a name="core-development-discussion"></a>
## Discurssão sobre o Desenvolvimento do Core

Discurssão sobre bugs, novos recursos, e implementação de funcionalidade existentes ocorre no canal do IRC `#laravel-dev`(Freenode). Taylor Otwell, o mantenedor to Laravel, é normalmente presente no canal nos dias de semana das 8h da manhã as 5 da tarde (UTC-06:00 or America/Chicago), e esporadicamente presente no canal em outros horários. 

O canal do IRC `#laravel-dev` é aberto da todos. Todos são bem vindo as se juntar o canal tanto seja para participar ou simplesmente observar as discussões. 

<a name="which-branch"></a>
## Qual Branch?

**Todas** as correções de bugs devem ser enviadas para o último branch estável. Correções de bugs  **nunca** devem ser enviadas para a branch `master` a menos que enviem correçoes de funcionalidades que existem no proxímo lançamento.  

**Menores** funcionalidade que são **totalmente compatíveis com as branchs anteriores**  e também com a branch atual do Laravel poderão ser enviada para última branch estável do Laravel.

**Major** new features should always be sent to the `master` branch, which contains the upcoming Laravel release.

If you are unsure if your feature qualifies as a major or minor, please ask Taylor Otwell in the `#laravel-dev` IRC channel (Freenode).

<a name="security-vulnerabilities"></a>
## Security Vulnerabilities

If you discover a security vulnerability within Laravel, please send an e-mail to Taylor Otwell at <a href="mailto:taylor@laravel.com">taylor@laravel.com</a>. All security vulnerabilities will be promptly addressed.

<a name="coding-style"></a>
## Coding Style

Laravel follows the [PSR-4](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-4-autoloader.md) and [PSR-1](https://github.com/php-fig/fig-standards/blob/master/accepted/PSR-1-basic-coding-standard.md) coding standards. In addition to these standards, the following coding standards should be followed:

- The class namespace declaration must be on the same line as `<?php`.
- A class' opening `{` must be on the same line as the class name.
- Functions and control structures must use Allman style braces.
- Indent with tabs, align with spaces.
