# Known Issues

## General
* wSAST can run out of memory when analyzing large projects usually resulting in an ApplicationException
	* Workaround: Divide and analyze piecemeal; increase RAM if using VM. 

## wSAST v0.1-alpha (release date 18-12-2023)
* The `calls from` feature occasionally produces nothing for a specified function
	* Workaround: Use the `paths` feature, so instead of `calls from .*?createUser.*` use `paths .*?createUser.* .*`

If you experience an issue and it is not here please email support@wsast.co.uk or message @wsastsupport on Twitter.

