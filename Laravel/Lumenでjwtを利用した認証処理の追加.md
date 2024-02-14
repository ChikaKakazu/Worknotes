# Lumenでjwtを利用した認証処理の追加
- Lumen10, php8.2で実装
- [lcobucci/jwt](https://github.com/lcobucci/jwt)を利用
    - [ドキュメント](https://lcobucci-jwt.readthedocs.io/en/latest/validating-tokens/)
- **jwt + セッション**の認証
- [JWT（JSON Web Token）認証の仕組みをやんわり理解してみる](https://neightbor.jp/blog/jsonwebtoken)

## jwtパッケージの導入
- jwtパッケージのインストール
    ```
    composer require lcobucci/jwt
    ```
- jwt作成で**期間**を利用する場合に**symfony/clock**をインストールする
    ```
    composer require symfony/clock
    ```

## jwtの作成・解析検証
- トークンの発行
    ```php
    /**
     * トークン発行
     *
     * @param User $user
     * @param string $auth_secret_key
     * @return string
     */
    public static function issuingToken(User $user, string $auth_secret_key): string
    {
        // 32文字のbase64エンコードされた文字列を投げる
        $key = InMemory::base64Encoded($auth_secret_key);

        // セッションIDの生成
        $salt = env('HASH_SALT');
        // jtiを生成する前情報
        $pre_jti = $user->uuid . Carbon::now()->timestamp;
        $jti = md5($pre_jti . $salt);
        $token = (new JwtFacade())->issue(
            new Sha256(),
            $key,
            static fn (
                Builder $builder,
                DateTimeImmutable $issue_at
            ) : Builder => $builder
                ->identifiedBy($jti, true)
                ->issuedAt($issue_at)
                ->expiresAt($issue_at->modify('+1 day'))
            );

        $user->pre_jti = $pre_jti;
        $user->save();

        return $token->toString();
    }
    ```
- トークンの解析と検証
    ```php
    /**
     * トークンの解析と検証
     * - トークンの構造
     * - トークンの有効期限
     * - トークンのセッションID
     * - トークンの署名
     *
     * @param Request $request
     * @param User $user
     * @param string $auth_secret_key
     * @return string
     */
    public static function parseAndValidateToken(Request $request, User $user, string $auth_secret_key): string
    {
        $jwt = $request->header('Request-Token');
        $parser = new Parser(new JoseEncoder());

        $validator = new Validator();
        $salt = env('HASH_SALT');

        try {
            // トークンの解析
            $token = $parser->parse($jwt);

            // トークンの検証
            $validator->assert($token, ...[
                new \Lcobucci\JWT\Validation\Constraint\LooseValidAt(new CarbonClock()),
                new \Lcobucci\JWT\Validation\Constraint\IdentifiedBy(md5($user->pre_jti . $salt)),
                new \Lcobucci\JWT\Validation\Constraint\SignedWith(new Sha256(), InMemory::base64Encoded($auth_secret_key)),
            ]);
        } catch (CannotDecodeContent | InvalidTokenStructure | UnsupportedHeaderFound $e) {
            // トークンの解析エラー
            throw app(ErrorService::class)->createException(
                app(ErrorService::class)::AUTH_PARSE_ERROR,
                [
                    'UserId' => $user->user_id,
                    'Uuid' => $user->uuid,
                    'UidHash' => $user->uid_hash,
                    'UidStatus' => $user->status,
                    'ParseError' => $e->getMessage(),
                ]
            );
        } catch (RequiredConstraintsViolated $e) {
            // トークンの検証エラー
            if (str_contains($e->violations()[0]->constraint, 'LooseValidAt') == true) {
                $validate_error = app(ErrorService::class)::AUTH_EXPIRED_ERROR;
            } elseif (str_contains($e->violations()[0]->constraint, 'IdentifiedBy') == true) {
                $validate_error = app(ErrorService::class)::AUTH_SESSION_ERROR;
            } elseif (str_contains($e->violations()[0]->constraint, 'SignedWith') == true) {
                $validate_error = app(ErrorService::class)::AUTH_SIGN_ERROR;
            } else {
                $validate_error = app(ErrorService::class)::AUTH_ERROR;
            }

            throw app(ErrorService::class)->createException(
                $validate_error,
                [
                    'UserId' => $user->user_id,
                    'Uuid' => $user->uuid,
                    'UidHash' => $user->uid_hash,
                    'UidStatus' => $user->status,
                    'ValidationError' => $e->getMessage(),
                ]
            );
        }

        return $jwt;
    }
    ```