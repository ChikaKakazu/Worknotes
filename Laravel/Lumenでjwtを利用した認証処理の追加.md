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
     * @param Request $request
     * @param User $user
     * @return \Illuminate\Http\JsonResponse
     */
    public function IssuingToken(Request $request, User $user): \Illuminate\Http\JsonResponse
    {
        $key = InMemory::base64Encoded('hiG8DlOKvtih6AxlZn5XKImZ06yu8I3mkOzaJrEuW8yAv8Jnkw330uMt8AEqQ5LB');
        // セッションIDの生成(userId + 有効期限 + 乱数とかで生成するのが良いかも)
        $session_id = bin2hex(random_bytes(16));
        // トークンの発行
        $token = (new JwtFacade())->issue(
            new Sha256(),
            $key,
            static fn (
                Builder $builder,
                DateTimeImmutable $issue_at
            ) : Builder => $builder
                // ここでトークンに付けたいパラメータを追加する
                ->issuedBy('http://localhost:8000')
                ->permittedFor('http://localhost:8000')
                ->identifiedBy($session_id, true)
                ->issuedAt($issue_at)
                ->expiresAt($issue_at->modify('+1 minute'))
            );

        // セッションIDを保存する
        $user->remember_token = $session_id;
        $user->save();

        return response()->json([
            'message' => 'success IssuingToken',
            'token' => $token->toString(),
        ]);
    }
    ```
- トークンの解析と検証
    ```php
    /**
     * トークンの解析と検証
     * - トークンの構造
     * - トークンの有効期限
     * - トークンのセッションID
     *
     * @param Request $request
     * @return \Illuminate\Http\JsonResponse
     */
    public function parseAndValidateToken(Request $request, User $user): \Illuminate\Http\JsonResponse
    {
        // トークンをhederから取り出す
        $jwt = $request->bearerToken();
        // トークンの解析(jwtの形になっているか)
        $parser = new Parser(new JoseEncoder());
        try {
            $token = $parser->parse($jwt);
        } catch (CannotDecodeContent | InvalidTokenStructure | UnsupportedHeaderFound $e) {
            return response()->json([
                'parse error' => $e->getMessage(),
            ], 401);
        }
        assert($token instanceof UnencryptedToken);

        // トークンの検証(有効期限や保存したセッションIDと一致しているかなど)
        $validator = new Validator();
        try {
            $validator->assert($token, ...[
                new \Lcobucci\JWT\Validation\Constraint\IssuedBy('http://localhost:8000'),
                new \Lcobucci\JWT\Validation\Constraint\PermittedFor('http://localhost:8000'),
                new \Lcobucci\JWT\Validation\Constraint\LooseValidAt(new NativeClock()),// 有効期限の検証
                new \Lcobucci\JWT\Validation\Constraint\IdentifiedBy($user->remember_token),// セッションIDの検証
            ]);
        } catch (RequiredConstraintsViolated $e) {
            $user->remember_token = '';
            $user->save();
            return response()->json([
                'validate error' => $e->getMessage(),
            ], 401);
        }

        return response()->json([
            'message' => 'success parseAndValidateToken',
            'token' => $token->toString(),
        ]);
    }
    ```